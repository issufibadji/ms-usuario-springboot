# usuario-api

Microserviço RESTful responsável pelo gerenciamento de usuários, endereços e telefones dentro do ecossistema **BFF Agendador de Tarefas**. É consumido exclusivamente pelo BFF via Feign Client HTTP e persiste dados em PostgreSQL.

## Posição na arquitetura

```text
Angular SPA (:4200)
      │
      ▼ HTTP/REST
BFF Agendador de Tarefas
      │
      ├──► usuario-api  (:8080)  ◄── este serviço → PostgreSQL (:5432)
      ├──► agendador    (:8090?)              → MongoDB (:27017)
      └──► notificacao                        → ViaCEP (externo)
```

O BFF é o único cliente direto desta API. Ele se autentica com JWT e repassa as chamadas vindas do front-end Angular para os microserviços corretos.

Veja a documentação completa do ecossistema em [docs/arquitetura-geral.md](docs/arquitetura-geral.md).

## Tecnologias

| Tecnologia | Versão |
| --- | --- |
| Java | 26 |
| Spring Boot | 3.5.0 |
| Spring Security + JWT | (via Boot / jjwt 0.12.6) |
| Spring Cloud OpenFeign | 2024.0.1 |
| PostgreSQL | 15+ |
| Springdoc OpenAPI | 2.8.9 |
| Gradle | 8.13 |

## Pré-requisitos

- Java 26+
- PostgreSQL 15+ rodando localmente
- Gradle 8.13+ (ou usar o wrapper incluído `./gradlew`)

## Configuração do banco de dados

Crie o banco antes de rodar a aplicação:

```sql
CREATE DATABASE db_usuario;
```

As credenciais padrão estão em `src/main/resources/application.properties`:

```properties
spring.datasource.url=jdbc:postgresql://localhost:5432/db_usuario
spring.datasource.username=postgres
spring.datasource.password=1234
```

Ajuste conforme o seu ambiente local.

## Executando localmente

```bash
./gradlew bootRun
```

A API estará disponível em <http://localhost:8080>.

Para gerar o JAR e executar diretamente:

```bash
./gradlew build
java -jar build/libs/usuario-0.0.1-SNAPSHOT.jar
```

## Documentação interativa (Swagger)

Acesse <http://localhost:8080/swagger-ui.html> para explorar todos os endpoints.

> Em produção, o Swagger é acessado pelo BFF ou equipe de desenvolvimento — não é exposto diretamente ao usuário final.

## Endpoints principais

| Método | Rota | Descrição | Auth |
| --- | --- | --- | --- |
| POST | `/usuario` | Criar usuário | Público |
| POST | `/usuario/login` | Autenticar e obter token JWT | Público |
| GET | `/usuario?email=` | Buscar usuário por e-mail | JWT |
| PUT | `/usuario` | Atualizar dados do usuário | JWT |
| DELETE | `/usuario/{email}` | Remover usuário | JWT |
| POST | `/usuario/endereco` | Adicionar endereço | JWT |
| PUT | `/usuario/endereco?id=` | Atualizar endereço | JWT |
| GET | `/usuario/endereco/{cep}` | Consultar CEP (ViaCEP) | Público |
| POST | `/usuario/telefone` | Adicionar telefone | JWT |
| PUT | `/usuario/telefone?id=` | Atualizar telefone | JWT |

O BFF chama estes endpoints passando o token JWT recebido do Angular no header `Authorization`.

Consulte [docs/endpoints.md](docs/endpoints.md) para detalhes de payloads e respostas.

## Autenticação

A API usa **JWT Bearer Token** com sessão `STATELESS`. O fluxo típico no ecossistema:

1. Angular envia credenciais ao BFF
2. BFF chama `POST /usuario/login` neste serviço
3. Este serviço valida e retorna o token JWT
4. BFF repassa o token ao Angular
5. Nas chamadas seguintes, o Angular envia o token → BFF o inclui no header → este serviço valida

```http
Authorization: Bearer <token>
```

Tokens expiram em **1 hora**.

## Usuários padrão (seed)

Ao iniciar com banco vazio, dois usuários são criados automaticamente:

| E-mail | Senha |
| --- | --- |
| `admin@admin.com` | `1234` |
| `usuario@email.com` | `usuario123$` |

## Estrutura do projeto

```text
src/main/java/com/issufibadji/usuario/
├── controller/          # Endpoints REST consumidos pelo BFF
├── business/            # Lógica de negócio e DTOs
│   └── dto/
├── infrastructure/
│   ├── entity/          # Entidades JPA (Usuario, Endereco, Telefone)
│   ├── repository/      # Repositórios Spring Data
│   ├── security/        # JWT, filtros e configuração de segurança
│   ├── clients/         # Feign client para ViaCEP
│   └── exceptions/      # Exceções customizadas e handler global
```

## Documentação completa

| Documento | Conteúdo |
| --- | --- |
| [docs/arquitetura-geral.md](docs/arquitetura-geral.md) | Ecossistema BFF, camadas internas, fluxos |
| [docs/endpoints.md](docs/endpoints.md) | Todos os endpoints com payloads |
| [docs/seguranca.md](docs/seguranca.md) | JWT, Spring Security, fluxo de auth |
| [docs/modelo-dados.md](docs/modelo-dados.md) | Entidades JPA, ERD, DTOs |
| [docs/integracoes.md](docs/integracoes.md) | BFF consumer, ViaCEP |
| [docs/infraestrutura.md](docs/infraestrutura.md) | Banco de dados, CI/CD, SonarQube |

## CI/CD

Pull Requests para `master` disparam build + testes automáticos via GitHub Actions (JDK 26).

## Qualidade de código

```bash
./gradlew sonarqube
```

> Requer instância SonarQube em <http://localhost:9000>.

## Licença

Este projeto é de uso educacional/demonstrativo.
