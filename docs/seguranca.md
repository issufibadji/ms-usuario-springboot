# Segurança

## Modelo de autenticação

A API utiliza **JWT (JSON Web Token)** com algoritmo **HMAC-SHA** para autenticação stateless. Nenhuma sessão é armazenada no servidor.

O chamador principal é o **BFF Agendador de Tarefas**, que recebe o token do Angular e o repassa nesta API via header `Authorization`.

## Componentes de segurança

### JwtUtil

Centraliza toda a lógica JWT:

- `generateToken(email)` — cria token com 1 hora de expiração
- `validateToken(token, userDetails)` — valida assinatura e expiração
- `extractEmail(token)` — extrai o subject (e-mail) do payload

### JwtRequestFilter

Filtro `OncePerRequestFilter` que intercepta cada requisição:

1. Lê o header `Authorization: Bearer {TOKEN}`
2. Extrai e valida o token via `JwtUtil`
3. Carrega o usuário via `UserDetailsServiceImpl`
4. Popula o `SecurityContextHolder`

### UserDetailsServiceImpl

Implementa `UserDetailsService` do Spring Security:

- Busca o usuário no banco pelo e-mail
- A entidade `Usuario` implementa `UserDetails` (roles, credenciais)

### SecurityConfig

Configura o `SecurityFilterChain`:

```text
Rotas públicas (sem token):
  POST /usuario                  — cadastro
  POST /usuario/login            — login
  GET  /usuario/endereco/{cep}   — consulta ViaCEP
  /swagger-ui/**                 — documentação
  /v3/api-docs/**                — spec OpenAPI

Todas as demais rotas: requerem JWT válido
```

Sessão configurada como `STATELESS` — nenhum cookie ou sessão HTTP é criado.

## Fluxo de autenticação (Angular → BFF → usuario-api)

```text
Angular
  |  POST /auth/login  { email, senha }
  v
BFF
  |  POST /usuario/login  { email, senha }
  v
usuario-api
  |  BCrypt.verify(senha)
  |  JwtUtil.generate(email)
  v
JWT Token
  |
  v  retorna ao BFF → retorna ao Angular
```

## Fluxo de autorização (BFF → usuario-api)

```text
Angular  -->  BFF  -->  usuario-api
                          |
                   JwtRequestFilter
                     1. extrai token do header
                     2. valida assinatura (JwtUtil)
                     3. carrega usuário (UserDetailsServiceImpl)
                     4. popula SecurityContextHolder
                          |
                   UsuarioController
```

## Configurações JWT

| Parâmetro | Valor |
| --- | --- |
| Algoritmo | HMAC-SHA |
| Expiração | 1 hora (3.600.000 ms) |
| Subject | E-mail do usuário |
| Chave | Base64 (configurada em `JwtUtil`) |

## Senhas

Todas as senhas são armazenadas com hash **BCrypt** via `BCryptPasswordEncoder`. Nenhuma senha é persistida em texto plano.

## Pontos de atenção

- A secret key JWT está hardcoded em `JwtUtil.java` — em produção deve ser externalizada via variável de ambiente ou secret manager.
- O token SonarQube em `build.gradle` deve ser movido para variável de ambiente (`SONAR_TOKEN`).
- O BFF deve validar o token antes de repassá-lo para evitar chamadas desnecessárias a esta API.
