# Integrações

## BFF Agendador de Tarefas (consumidor desta API)

### Papel do BFF

O **BFF Agendador de Tarefas** é o único cliente direto da `usuario-api`. O front-end Angular nunca chama esta API diretamente — toda comunicação é intermediada pelo BFF.

```text
Angular SPA
    │  requisição + JWT
    ▼
BFF Agendador de Tarefas
    │  Feign Client + Authorization: Bearer {TOKEN}
    ▼
usuario-api (:8080)
    │
    ▼
PostgreSQL (:5432)
```

### Como o BFF se conecta

O BFF usa **Spring Cloud OpenFeign** para declarar um cliente HTTP tipado que aponta para esta API:

```java
// No projeto BFF
@FeignClient(name = "usuario-api", url = "${usuario.api.url}")
public interface UsuarioClient {
    @PostMapping("/usuario/login")
    String login(@RequestBody UsuarioDTO usuario);

    @GetMapping("/usuario")
    UsuarioDTO buscarPorEmail(@RequestParam String email,
                              @RequestHeader("Authorization") String token);
    // demais endpoints...
}
```

### Fluxo de autenticação no ecossistema

```text
1. Angular  →  POST /auth/login  →  BFF
2. BFF      →  POST /usuario/login { email, senha }  →  usuario-api
3. usuario-api valida credenciais e gera JWT
4. JWT retorna: usuario-api → BFF → Angular
5. Angular armazena o token e o envia em todas as chamadas seguintes
6. BFF repassa o token no header Authorization para cada microserviço
```

### Endpoints consumidos pelo BFF

| Endpoint | Uso no BFF |
| --- | --- |
| `POST /usuario/login` | Autenticação do usuário |
| `POST /usuario` | Cadastro de novo usuário |
| `GET /usuario?email=` | Buscar dados do usuário logado |
| `PUT /usuario` | Atualizar perfil |
| `DELETE /usuario/{email}` | Remover conta |
| `POST /usuario/endereco` | Adicionar endereço |
| `PUT /usuario/endereco?id=` | Atualizar endereço |
| `GET /usuario/endereco/{cep}` | Consultar CEP (repassado do Angular) |
| `POST /usuario/telefone` | Adicionar telefone |
| `PUT /usuario/telefone?id=` | Atualizar telefone |

---

## ViaCEP (API externa)

### O que é

[ViaCEP](https://viacep.com.br) é um serviço gratuito e público para consulta de endereços a partir do CEP brasileiro.

### Implementação

A integração usa **Spring Cloud OpenFeign** — cliente HTTP declarativo:

```java
@FeignClient(name = "viacep", url = "${viacep.url}")
public interface ViaCepClient {
    @GetMapping("/ws/{cep}/json/")
    ViaCepDTO buscarEndereco(@PathVariable String cep);
}
```

```properties
viacep.url=https://viacep.com.br
```

### Fluxo de consulta

```text
BFF / Angular  →  GET /usuario/endereco/{cep}
               →  ViaCepService.buscarEndereco(cep)
                   1. Valida formato (8 dígitos numéricos)
                   2. Feign → GET https://viacep.com.br/ws/{cep}/json/
                   3. Resposta deserializada em ViaCepDTO
               ←  ViaCepDTO retornado ao BFF
```

### Resposta da API ViaCEP

```json
{
  "cep": "01310-100",
  "logradouro": "Praça da Sé",
  "complemento": "lado ímpar",
  "bairro": "Sé",
  "localidade": "São Paulo",
  "uf": "SP",
  "ibge": "3550308",
  "ddd": "11"
}
```

### Limitações

| Item | Detalhe |
| --- | --- |
| Rate limit | Não documentado — uso excessivo pode gerar bloqueio |
| Disponibilidade | Serviço externo sem SLA |
| Cache | Não implementado — cada chamada faz uma requisição HTTP |
| Timeout | Configurável via propriedades do Feign |

**Melhoria recomendada:** adicionar `@Cacheable` no `ViaCepService` para evitar chamadas repetidas ao mesmo CEP.

---

## Resumo de dependências externas

| Serviço | Tipo | Endpoint | Auth |
| --- | --- | --- | --- |
| BFF Agendador de Tarefas | Consumidor (Feign Client) | `:8080` | JWT Bearer |
| ViaCEP | REST API pública | `https://viacep.com.br/ws/{cep}/json/` | Nenhuma |
| PostgreSQL | Banco de dados | `localhost:5432` | usuário/senha |
