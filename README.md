# usuario-api

API RESTful para gerenciamento de usuários, endereços e telefones, construída com Spring Boot 3.5, Java 26 e autenticação JWT. Integra-se com o serviço ViaCEP para consulta de endereços por CEP.

## Tecnologias

| Tecnologia | Versão |
| --- | --- |
| Java | 26 |
| Spring Boot | 3.5.0 |
| Spring Security | (via Boot) |
| Spring Cloud OpenFeign | 2024.0.1 |
| PostgreSQL | 15+ |
| JWT (jjwt) | 0.12.6 |
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

Acesse <http://localhost:8080/swagger-ui.html> para explorar todos os endpoints via interface gráfica.

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

Consulte [docs/endpoints.md](docs/endpoints.md) para detalhes completos de payloads e respostas.

## Autenticação

A API usa **JWT Bearer Token**. Após fazer login, inclua o token no header de todas as requisições protegidas:

```http
Authorization: Bearer <token>
```

Os tokens expiram em **1 hora**.

## Usuários padrão (seed)

Ao iniciar pela primeira vez com banco vazio, dois usuários são criados automaticamente:

| E-mail | Senha |
| --- | --- |
| `admin@admin.com` | `1234` |
| `usuario@email.com` | `usuario123$` |

## Estrutura do projeto

```text
src/main/java/com/issufibadji/usuario/
├── controller/          # Endpoints REST
├── business/            # Lógica de negócio e DTOs
│   └── dto/
├── infrastructure/
│   ├── entity/          # Entidades JPA (Usuario, Endereco, Telefone)
│   ├── repository/      # Repositórios Spring Data
│   ├── security/        # JWT, filtros e configuração de segurança
│   ├── clients/         # Feign client para ViaCEP
│   └── exceptions/      # Exceções customizadas e handler global
```

Veja a documentação completa em [docs/](docs/).

## CI/CD

Pull Requests para `master` disparam build + testes automáticos via GitHub Actions (JDK 26).

## Qualidade de código

Análise estática configurada com **SonarQube**:

```bash
./gradlew sonarqube
```

> Requer instância SonarQube rodando em <http://localhost:9000>.

## Licença

Este projeto é de uso educacional/demonstrativo.
