# Tech Stack — Sistema Interno da Loja

## 1. Backend
- **Node.js**
- **Express.js**
- Estrutura REST API
- Middlewares para autenticação e autorização
- Validação de dados com bibliotecas externas
- Controle de sessão por token (JWT ou sessão local)

---

## 2. Banco de Dados
- **PostgreSQL**
- Modelo relacional com tabelas para:
  - usuários
  - produtos
  - clientes
  - vendas
  - itens de venda
- Migrações de schema (opcional com Knex, Prisma ou Sequelize)

---

## 3. Frontend
- **React**
- Componentização das telas:
  - Login
  - Produtos
  - Clientes
  - Vendas
  - Dashboards (admin)
- Comunicação com API via HTTP (fetch ou axios)

---

## 4. Deploy e Infraestrutura
- **Docker**
  - Um container para o Backend (Node + Express)
  - Um container para o PostgreSQL
  - Rede interna Docker entre os containers
- **docker-compose**
  - Orquestração de app + banco
  - Volume persistente para os dados do PostgreSQL
