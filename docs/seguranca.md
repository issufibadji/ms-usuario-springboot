# Segurança

## Modelo de autenticação

A API utiliza **JWT (JSON Web Token)** com algoritmo **HMAC-SHA** para autenticação stateless. Nenhuma sessão é armazenada no servidor.

## Componentes de segurança

### JwtUtil
Centraliza toda a lógica JWT:
- `generateToken(email)` — cria token com 1 hora de expiração
- `validateToken(token, userDetails)` — valida assinatura e expiração
- `extractEmail(token)` — extrai o subject (e-mail) do payload

### JwtRequestFilter
Filtro `OncePerRequestFilter` que intercepta cada requisição:
1. Lê o header `Authorization: Bearer <token>`
2. Extrai e valida o token via `JwtUtil`
3. Carrega o usuário via `UserDetailsServiceImpl`
4. Popula o `SecurityContextHolder`

### UserDetailsServiceImpl
Implementa `UserDetailsService` do Spring Security:
- Busca o usuário no banco pelo e-mail
- A entidade `Usuario` implementa `UserDetails` (roles, credenciais)

### SecurityConfig
Configura o `SecurityFilterChain`:

```
Rotas públicas (sem token):
  POST /usuario          — cadastro
  POST /usuario/login    — login
  GET  /usuario/endereco/{cep}  — consulta ViaCEP
  /swagger-ui/**         — documentação
  /v3/api-docs/**        — spec OpenAPI

Todas as demais rotas: requerem JWT válido
```

Sessão configurada como `STATELESS` — nenhum cookie ou sessão HTTP é criado.

## Fluxo de autenticação

```
┌──────────┐     POST /usuario/login      ┌─────────────┐
│  Cliente │  ──────────────────────────► │  Controller │
└──────────┘    { email, senha }          └──────┬──────┘
                                                 │
                                          ┌──────▼──────┐
                                          │   Service   │
                                          │  BCrypt.verify()
                                          │  JwtUtil.generate()
                                          └──────┬──────┘
                                                 │
┌──────────┐   ◄── { token: "eyJ..." }  ┌──────▼──────┐
│  Cliente │                            │  Resposta   │
└──────────┘                            └─────────────┘
```

## Fluxo de autorização

```
┌──────────┐   GET /usuario?email=X        ┌─────────────────────┐
│  Cliente │  Authorization: Bearer eyJ... │  JwtRequestFilter   │
└──────────┘  ──────────────────────────► │  1. extrai token     │
                                           │  2. valida assinatura│
                                           │  3. carrega usuário  │
                                           │  4. seta contexto    │
                                           └──────────┬──────────┘
                                                      │
                                              ┌───────▼──────────┐
                                              │   Controller     │
                                              └──────────────────┘
```

## Configurações JWT

| Parâmetro | Valor |
|---|---|
| Algoritmo | HMAC-SHA |
| Expiração | 1 hora (3.600.000 ms) |
| Subject | E-mail do usuário |
| Chave | Base64 (configurada em `JwtUtil`) |

## Senhas

Todas as senhas são armazenadas com hash **BCrypt** via `BCryptPasswordEncoder`. Nenhuma senha é persistida em texto plano.

## Vulnerabilidades conhecidas / pontos de atenção

- A secret key JWT está hardcoded em `JwtUtil.java`. Em produção, deve ser externalizada via variável de ambiente ou secret manager.
- O token SonarQube em `build.gradle` deve ser movido para variável de ambiente (`SONAR_TOKEN`).
