# Infraestrutura

## Banco de dados

A aplicação requer **PostgreSQL 15+** rodando localmente.

### Criação do banco

```sql
CREATE DATABASE db_usuario;
```

### Configurações de conexão

Definidas em `src/main/resources/application.properties`:

```properties
spring.datasource.url=jdbc:postgresql://localhost:5432/db_usuario
spring.datasource.username=postgres
spring.datasource.password=1234
spring.jpa.hibernate.ddl-auto=update
spring.jpa.show-sql=true
```

O DDL `update` cria ou atualiza as tabelas automaticamente na primeira execução.

---

## Executando a aplicação

### Com Gradle wrapper (recomendado para desenvolvimento)

```bash
./gradlew bootRun
```

### Gerando e executando o JAR

```bash
./gradlew build
java -jar build/libs/usuario-0.0.1-SNAPSHOT.jar
```

A API ficará disponível em <http://localhost:8080>.

---

## CI/CD — GitHub Actions

### Workflow: PR (`gradle.yml`)

**Gatilho:** Pull Request para a branch `master`

**Passos:**

1. Checkout do código
2. Setup JDK 26 (Temurin)
3. Cache de dependências Gradle
4. `./gradlew build` — compila e roda testes

---

## Qualidade de código — SonarQube

Análise estática configurada via plugin Gradle:

```bash
./gradlew sonarqube
```

| Parâmetro | Valor |
| --- | --- |
| Project Key | usuario |
| Host | <http://localhost:9000> |
| Token | variável `SONAR_TOKEN` (não hardcode) |

---

## Configuração de ambiente

Todas as propriedades sensíveis (senha do banco, secret JWT, token Sonar) devem ser sobrescritas por variáveis de ambiente ou por um arquivo `application-local.properties` ignorado pelo `.gitignore`.

### Exemplo com variáveis de ambiente

```bash
export SPRING_DATASOURCE_PASSWORD=minha_senha_segura
./gradlew bootRun
```
