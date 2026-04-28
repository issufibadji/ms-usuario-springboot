# Endpoints da API

Base URL: `http://localhost:8080`

Documentação interativa: `http://localhost:8080/swagger-ui.html`

---

## Autenticação

### POST `/usuario/login`
Autentica o usuário e retorna um JWT.

**Público — sem token.**

**Request body:**
```json
{
  "email": "usuario@email.com",
  "senha": "usuario123$"
}
```

**Response 200:**
```json
"eyJhbGciOiJIUzI1NiJ9..."
```

**Erros:**
| Status | Motivo |
|---|---|
| 401 | Credenciais inválidas |

---

## Usuários

### POST `/usuario`
Cria um novo usuário.

**Público — sem token.**

**Request body:**
```json
{
  "nome": "João Silva",
  "email": "joao@email.com",
  "senha": "minhasenha123",
  "enderecos": [],
  "telefones": []
}
```

**Response 200:** objeto `UsuarioDTO` criado.

**Erros:**
| Status | Motivo |
|---|---|
| 409 | E-mail já cadastrado |

---

### GET `/usuario?email={email}`
Busca um usuário pelo e-mail.

**Requer token JWT.**

**Response 200:**
```json
{
  "nome": "João Silva",
  "email": "joao@email.com",
  "enderecos": [...],
  "telefones": [...]
}
```

**Erros:**
| Status | Motivo |
|---|---|
| 404 | Usuário não encontrado |

---

### PUT `/usuario`
Atualiza dados do usuário autenticado. O e-mail é extraído do token JWT — não é necessário informá-lo no body.

**Requer token JWT.**

**Request body** (campos opcionais — apenas os informados são atualizados):
```json
{
  "nome": "João Atualizado",
  "senha": "novasenha456"
}
```

**Response 200:** objeto `UsuarioDTO` atualizado.

---

### DELETE `/usuario/{email}`
Remove o usuário pelo e-mail.

**Requer token JWT.**

**Response 200:** sem body.

**Erros:**
| Status | Motivo |
|---|---|
| 404 | Usuário não encontrado |

---

## Endereços

### POST `/usuario/endereco`
Adiciona um endereço ao usuário autenticado.

**Requer token JWT.**

**Request body:**
```json
{
  "rua": "Rua das Flores",
  "numero": "123",
  "complemento": "Apto 4",
  "cidade": "São Paulo",
  "estado": "SP",
  "cep": "01310100"
}
```

**Response 200:** objeto `UsuarioDTO` atualizado com o novo endereço.

---

### PUT `/usuario/endereco?id={id}`
Atualiza um endereço existente pelo ID.

**Requer token JWT.**

**Request body:** campos do endereço (os não informados não são alterados).

**Response 200:** objeto `EnderecoDTO` atualizado.

---

### GET `/usuario/endereco/{cep}`
Consulta um endereço pelo CEP via ViaCEP.

**Público — sem token.**

**Path param:** `cep` — 8 dígitos numéricos (ex: `01310100`)

**Response 200:**
```json
{
  "cep": "01310-100",
  "logradouro": "Praça da Sé",
  "bairro": "Sé",
  "localidade": "São Paulo",
  "uf": "SP"
}
```

**Erros:**
| Status | Motivo |
|---|---|
| 400 | CEP inválido (formato incorreto) |

---

## Telefones

### POST `/usuario/telefone`
Adiciona um telefone ao usuário autenticado.

**Requer token JWT.**

**Request body:**
```json
{
  "numero": "912345678",
  "ddd": "11"
}
```

**Response 200:** objeto `UsuarioDTO` atualizado com o novo telefone.

---

### PUT `/usuario/telefone?id={id}`
Atualiza um telefone existente pelo ID.

**Requer token JWT.**

**Request body:** campos do telefone.

**Response 200:** objeto `TelefoneDTO` atualizado.

---

## Estrutura de erros

Todos os erros retornam o seguinte formato:

```json
{
  "timestamp": "2026-04-28T12:00:00",
  "status": 404,
  "error": "Not Found",
  "message": "Usuário não encontrado",
  "path": "/usuario"
}
```
