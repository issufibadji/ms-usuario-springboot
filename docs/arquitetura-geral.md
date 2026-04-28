# Arquitetura Geral

## Visão geral

A **usuario-api** segue uma arquitetura em camadas clássica (**Layered Architecture**), com separação explícita entre apresentação, negócio e infraestrutura. Toda comunicação externa passa por uma camada de segurança baseada em JWT.

```
┌──────────────────────────────────────────────────┐
│                   Cliente HTTP                   │
└────────────────────┬─────────────────────────────┘
                     │ HTTP/REST
┌────────────────────▼─────────────────────────────┐
│             Spring Security (JWT Filter)          │
└────────────────────┬─────────────────────────────┘
                     │
┌────────────────────▼─────────────────────────────┐
│               Controller Layer                    │
│         (UsuarioController)                       │
└────────────────────┬─────────────────────────────┘
                     │
┌────────────────────▼─────────────────────────────┐
│               Business Layer                      │
│       (UsuarioService, ViaCepService)             │
└──────────┬─────────────────────┬─────────────────┘
           │                     │
┌──────────▼──────────┐  ┌───────▼───────────────┐
│  Repository Layer   │  │  External Clients      │
│ (JPA Repositories)  │  │  (ViaCEP via Feign)    │
└──────────┬──────────┘  └───────────────────────┘
           │
┌──────────▼──────────┐
│     PostgreSQL       │
└─────────────────────┘
```

## Camadas

### Controller
- Única classe: `UsuarioController`
- Responsável por receber requisições HTTP, validar entrada e devolver respostas
- Não contém lógica de negócio — delega tudo ao serviço

### Business (Service)
- `UsuarioService` — CRUD de usuários, endereços e telefones; geração e validação de JWT
- `ViaCepService` — validação de CEP e chamada ao cliente Feign; encapsula regras de formato

### Infrastructure
Dividida em subpacotes:

| Subpacote | Responsabilidade |
|---|---|
| `entity` | Entidades JPA mapeadas para o banco |
| `repository` | Interfaces Spring Data JPA |
| `security` | JWT, filtros, UserDetailsService, SecurityConfig |
| `clients` | Feign client para APIs externas |
| `exceptions` | Hierarquia de exceções e `@ControllerAdvice` global |

## Fluxo de uma requisição protegida

```
1. Cliente envia request com header Authorization: Bearer <token>
2. JwtRequestFilter intercepta a request
3. JwtUtil valida e extrai o e-mail do token
4. UserDetailsServiceImpl carrega o usuário do banco
5. SecurityContextHolder é populado
6. DispatcherServlet encaminha para UsuarioController
7. Controller chama UsuarioService
8. UsuarioService usa Repositories e/ou ViaCepService
9. Resposta retorna ao cliente
```

## Fluxo de autenticação (login)

```
1. POST /usuario/login { email, senha }
2. UsuarioService busca usuário por email
3. BCryptPasswordEncoder valida a senha
4. JwtUtil gera token com expiração de 1 hora
5. Token retornado no body da resposta
```

## Tratamento de erros

Todas as exceções são capturadas pelo `GlobalExceptionHandler` (`@ControllerAdvice`) que retorna um `ErrorResponseDTO` padronizado:

```json
{
  "timestamp": "2026-04-28T12:00:00",
  "status": 404,
  "error": "Not Found",
  "message": "Usuário não encontrado",
  "path": "/usuario"
}
```

| Exceção | HTTP Status |
|---|---|
| `ResourceNotFoundException` | 404 |
| `ConflictException` | 409 |
| `UnauthorizedException` | 401 |
| `IllegalArgumentException` | 400 |

## Tecnologias-chave

- **Spring Boot 3.5** — auto-configuração e gestão de ciclo de vida
- **Spring Data JPA** — abstração sobre Hibernate/PostgreSQL
- **Spring Security** — filtro JWT e configuração de rotas públicas/protegidas
- **Spring Cloud OpenFeign** — cliente HTTP declarativo para ViaCEP
- **Lombok** — redução de boilerplate (getters, setters, builders)
- **Springdoc OpenAPI 2.6** — geração automática de Swagger UI
