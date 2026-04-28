# Integrações Externas

## ViaCEP

### O que é

[ViaCEP](https://viacep.com.br) é um serviço gratuito e público brasileiro para consulta de endereços a partir do CEP (Código de Endereçamento Postal).

### Implementação

A integração usa **Spring Cloud OpenFeign** — um cliente HTTP declarativo que gera a implementação em tempo de compilação a partir de uma interface anotada.

**Interface Feign:**
```java
@FeignClient(name = "viacep", url = "${viacep.url}")
public interface ViaCepClient {
    @GetMapping("/ws/{cep}/json/")
    ViaCepDTO buscarEndereco(@PathVariable String cep);
}
```

**Configuração em `application.properties`:**
```properties
viacep.url=https://viacep.com.br
```

### Fluxo de consulta

```
Cliente  →  GET /usuario/endereco/{cep}
         →  ViaCepService.buscarEndereco(cep)
             1. Valida formato do CEP (8 dígitos)
             2. Chama ViaCepClient.buscarEndereco(cep)
             3. Feign faz GET https://viacep.com.br/ws/{cep}/json/
             4. Resposta deserializada em ViaCepDTO
         ←  ViaCepDTO retornado ao cliente
```

### Validação do CEP

Antes de chamar o serviço externo, o `ViaCepService` valida que o CEP:
- Contém exatamente 8 caracteres
- Contém apenas dígitos numéricos

Se inválido, lança `IllegalArgumentException` (HTTP 400).

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
  "gia": "1004",
  "ddd": "11",
  "siafi": "7107"
}
```

### DTO de mapeamento

```java
public class ViaCepDTO {
    private String cep;
    private String logradouro;
    private String complemento;
    private String bairro;
    private String localidade;
    private String uf;
    private String ddd;
    // ...outros campos
}
```

### Limitações e considerações

| Item | Detalhe |
|---|---|
| Rate limit | ViaCEP não documenta rate limit oficial, mas uso excessivo pode resultar em bloqueio |
| Disponibilidade | Serviço externo — sem SLA garantido |
| Cache | Não implementado; cada chamada faz uma requisição HTTP |
| Timeout | Configurável via propriedades do Feign |

**Melhoria recomendada:** adicionar cache (`@Cacheable`) no `ViaCepService` para evitar chamadas repetidas ao mesmo CEP.

---

## Dependências externas em geral

| Serviço | Tipo | Endpoint | Autenticação |
|---|---|---|---|
| ViaCEP | REST API | `https://viacep.com.br/ws/{cep}/json/` | Nenhuma (pública) |
| PostgreSQL | Banco de dados | `localhost:5432` | usuário/senha |
| SonarQube | Qualidade de código | `localhost:9000` | token |
