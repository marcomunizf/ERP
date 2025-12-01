# API Routes — Sistema ERP da Loja  
> Versão: 0.1  
> Backend: Node.js + Express  
> Banco: PostgreSQL  
> Autenticação: JWT (Bearer Token)

---

# 1. Convenções Gerais

- **Base URL:** `/api`
- Todas as rotas (exceto `/auth/login`) exigem:
  - `Authorization: Bearer <token>`
- Códigos de erro padrão:
  - `400 Bad Request` – dados inválidos
  - `401 Unauthorized` – falta de token ou token inválido
  - `403 Forbidden` – usuário sem permissão
  - `404 Not Found`
  - `500 Internal Server Error`

---

# 2. Autenticação

## 2.1. Login

POST /api/auth/login

Body:
{
  "usuario": "joao",
  "senha": "minhaSenha123"
}

Resposta 200:
{
  "token": "<jwt_token>",
  "usuario": {
    "id": 1,
    "nome": "João da Silva",
    "usuario": "joao",
    "perfil": "VENDEDOR",
    "ativo": true
  }
}

---

# 3. Usuários (Admin Only)

GET /api/users  
POST /api/users  
PUT /api/users/:id  
PATCH /api/users/:id/password  

---

# 4. Produtos

GET /api/products  
GET /api/products/:id  
POST /api/products  
PUT /api/products/:id  
PATCH /api/products/:id/deactivate  

---

# 5. Clientes

GET /api/clients  
GET /api/clients/:id  
POST /api/clients  
PUT /api/clients/:id  
PATCH /api/clients/:id/deactivate  

---

# 6. Vendas

GET /api/sales  
GET /api/sales/:id  
POST /api/sales  
PATCH /api/sales/:id/payment-status  

---

# 7. Clientes Devedores (Dívidas)

GET /api/debts/clients  
GET /api/debts/clients/:idCliente  

---

# 8. Dashboard (Admin)

GET /api/dashboard/summary  
GET /api/dashboard/sales-by-user  

---

# 9. Erros

400 Bad Request  
401 Unauthorized  
403 Forbidden  
404 Not Found  
500 Internal Server Error  
