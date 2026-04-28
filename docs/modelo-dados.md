# Modelo de Dados

## Diagrama de entidades (ERD simplificado)

```
┌─────────────────────┐       ┌──────────────────────┐
│       usuario       │       │       endereco        │
├─────────────────────┤       ├──────────────────────┤
│ id          BIGINT PK│──┐   │ id          BIGINT PK │
│ nome        VARCHAR │  │   │ rua         VARCHAR   │
│ email       VARCHAR │  │   │ numero      VARCHAR   │
│ senha       VARCHAR │  │   │ complemento VARCHAR   │
└─────────────────────┘  │   │ cidade      VARCHAR   │
                         │   │ estado      VARCHAR   │
                         ├──►│ cep         VARCHAR   │
                         │   │ usuario_id  BIGINT FK │
                         │   └──────────────────────┘
                         │
                         │   ┌──────────────────────┐
                         │   │       telefone        │
                         │   ├──────────────────────┤
                         │   │ id          BIGINT PK │
                         └──►│ numero      VARCHAR   │
                             │ ddd         VARCHAR   │
                             │ usuario_id  BIGINT FK │
                             └──────────────────────┘
```

## Entidade: Usuario

**Tabela:** `usuario`

| Coluna | Tipo | Restrições |
|---|---|---|
| id | BIGINT | PK, AUTO_INCREMENT |
| nome | VARCHAR(100) | NOT NULL |
| email | VARCHAR(100) | NOT NULL, UNIQUE |
| senha | VARCHAR | NOT NULL (hash BCrypt) |

**Relacionamentos:**
- `enderecos` — OneToMany com `Endereco` (cascade ALL, fetch LAZY)
- `telefones` — OneToMany com `Telefone` (cascade ALL, fetch LAZY)

**Implementa `UserDetails`** (Spring Security) — fornece `getUsername()` retornando o e-mail.

---

## Entidade: Endereco

**Tabela:** `endereco`

| Coluna | Tipo | Restrições |
|---|---|---|
| id | BIGINT | PK, AUTO_INCREMENT |
| rua | VARCHAR | — |
| numero | VARCHAR | — |
| complemento | VARCHAR | — |
| cidade | VARCHAR | — |
| estado | VARCHAR | — |
| cep | VARCHAR | — |
| usuario_id | BIGINT | FK → usuario.id |

**Relacionamento:** ManyToOne com `Usuario`.

---

## Entidade: Telefone

**Tabela:** `telefone`

| Coluna | Tipo | Restrições |
|---|---|---|
| id | BIGINT | PK, AUTO_INCREMENT |
| numero | VARCHAR | — |
| ddd | VARCHAR | — |
| usuario_id | BIGINT | FK → usuario.id |

**Relacionamento:** ManyToOne com `Usuario`.

---

## DTOs (Data Transfer Objects)

Os DTOs isolam o contrato da API das entidades JPA:

### UsuarioDTO
```json
{
  "nome": "string",
  "email": "string",
  "senha": "string",
  "enderecos": [ EnderecoDTO ],
  "telefones": [ TelefoneDTO ]
}
```

### EnderecoDTO
```json
{
  "id": 1,
  "rua": "string",
  "numero": "string",
  "complemento": "string",
  "cidade": "string",
  "estado": "string",
  "cep": "string"
}
```

### TelefoneDTO
```json
{
  "id": 1,
  "numero": "string",
  "ddd": "string"
}
```

### ViaCepDTO (resposta do serviço externo)
```json
{
  "cep": "string",
  "logradouro": "string",
  "complemento": "string",
  "bairro": "string",
  "localidade": "string",
  "uf": "string",
  "ddd": "string"
}
```

---

## Conversão Entity ↔ DTO

O componente `UsuarioConverter` centraliza todas as conversões:

- `paraUsuario(UsuarioDTO)` — converte DTO para entidade (para persistência)
- `paraUsuarioDTO(Usuario)` — converte entidade para DTO (para resposta)
- Atualizações parciais: campos `null` no DTO não sobrescrevem os valores existentes na entidade

---

## Configuração Hibernate

```properties
spring.jpa.hibernate.ddl-auto=update
spring.jpa.show-sql=true
```

O DDL `update` cria ou atualiza tabelas automaticamente. Em produção, recomenda-se usar Flyway ou Liquibase para migrações controladas.
