# architecture.md  
Sistema Interno da Loja — Cadastro de Produtos, Clientes, Vendas e Dívidas

---

## 1. Visão Geral

Este documento descreve a arquitetura do sistema interno da loja utilizado por **vendedores** e **admins** para:

- Cadastrar produtos e clientes.
- Registrar vendas de produtos para clientes.
- Controlar clientes que possuem dívidas em aberto.
- Permitir a admins o acesso a dashboards e gestão de usuários (vendedores).

O sistema será uma **aplicação multiusuário**, com autenticação por login e controle de permissões por perfil.

---

## 2. Objetivos da Arquitetura

- Manter a **organização dos dados** da loja (produtos, clientes, vendas, dívidas).
- Fornecer **fluxos simples** para o vendedor utilizar no dia a dia.
- Garantir que **admins** tenham uma visão consolidada em **dashboards**.
- Separar responsabilidades entre camadas (frontend, backend, banco de dados).
- Permitir **evolução futura** (ex.: relatórios financeiros mais complexos, estoque, integração com outros sistemas).

### Fora de escopo (por enquanto)

- Integração com sistemas de pagamento externos (cartão, PIX, etc.).
- Controle detalhado de estoque e nota fiscal.
- Multi-loja ou multi-filial.
- App mobile nativo (Android/iOS).

---

## 3. Contexto do Sistema

### 3.1. Atores

- **Vendedor**
  - Autentica no sistema.
  - Cadastra produtos (se permitido pela regra de negócio) ou usa produtos existentes.
  - Cadastra clientes.
  - Registra vendas.
  - Consulta lista de clientes com dívidas em aberto.

- **Admin**
  - Faz tudo o que o vendedor faz.
  - Cria e exclui vendedores (gestão de usuários).
  - Acessa telas específicas de **dashboard** (indicadores e visão consolidada).

### 3.2. Diagrama de Contexto (Descrição Textual)

- Usuários (Vendedor/Admin) acessam o **Frontend** (navegador ou app desktop).
- O Frontend se comunica com uma **API Backend**.
- A API Backend acessa um **Banco de Dados Relacional** para ler/escrever:
  - Produtos
  - Clientes
  - Vendas
  - Usuários
  - Perfis (role: vendedor/admin)
  - Informações de dívida (valores em aberto)

---

## 4. Visão de Usuários e Permissões

### 4.1. Perfis

- **Vendedor**
  - Pode: logar, cadastrar cliente, cadastrar venda, consultar produtos, consultar clientes devedores.
  - Pode (opcional, definível em regra): cadastrar produtos.
  - Não pode: criar/excluir usuários, acessar dashboards administrativos.

- **Admin**
  - Pode: tudo que o vendedor pode.
  - Pode: criar, editar e excluir vendedores.
  - Pode: acessar dashboards e relatórios administrativos.

### 4.2. Controle de Acesso

- Autenticação por **login/senha**.
- Autorização baseada em **role** (`VENDEDOR` / `ADMIN`) armazenada no banco.
- Rotas e funcionalidades protegidas no Backend (ex.: APIs de dashboard e gestão de usuários só aceitam `ADMIN`).

---

## 5. Arquitetura Lógica

### 5.1. Camadas Principais

1. **Camada de Apresentação (Frontend)**
   - Interface para login.
   - Telas de:
     - Cadastro de produtos.
     - Cadastro de clientes.
     - Registro de vendas.
     - Lista de clientes devedores.
     - Dashboards (apenas admin).
   - Comunicação com backend via HTTP/JSON (ex.: REST API).

2. **Camada de Negócio (Backend / API)**
   - Exposição de endpoints para:
     - Autenticação.
     - CRUD de produtos.
     - CRUD de clientes.
     - Registro de vendas.
     - Consultas de dívidas.
     - Gestão de usuários (apenas admin).
     - Consultas de métricas para dashboards.
   - Implementação de regras de negócio (ex.: cálculo de total da venda, marcação de dívida em aberto, validação de acesso por perfil).

3. **Camada de Persistência (Banco de Dados)**
   - Banco de dados relacional.
   - Tabelas para usuários, produtos, clientes, vendas, itens da venda, status de pagamento, etc.
   - Garantia de integridade referencial via chaves estrangeiras.

---

## 6. Módulos / Domínios Funcionais

### 6.1. Módulo de Autenticação e Usuários

- Entidades:
  - `Usuario`
  - `Perfil` (role: `ADMIN`, `VENDEDOR`)
- Funções:
  - Login.
  - Criação, edição e exclusão de usuários (apenas admin).
  - Alteração de senha (opcional).

### 6.2. Módulo de Produtos

- Entidade: `Produto`
- Funções:
  - Criar, editar, listar e inativar produtos.
  - Validações básicas (nome obrigatório, preço >= 0).

### 6.3. Módulo de Clientes

- Entidade: `Cliente`
- Funções:
  - Criar, editar, listar clientes.
  - Campos básicos como nome, documento, contato.

### 6.4. Módulo de Vendas

- Entidades:
  - `Venda`
  - `ItemVenda`
- Funções:
  - Registrar venda de um ou mais produtos para um cliente.
  - Calcular total da venda.
  - Associar venda a um vendedor.
  - Marcar status de pagamento:
    - `PAGO`
    - `EM_ABERTO`
    - (opcional) `PARCIAL`

### 6.5. Módulo de Dívidas / Contas a Receber

- Baseado em `Venda` + status de pagamento.
- Funções:
  - Listar clientes com vendas `EM_ABERTO`.
  - Mostrar total em aberto por cliente.
  - Exibir histórico de vendas pendentes.

### 6.6. Módulo de Dashboards (Admin)

- Funções:
  - Indicadores gerais de vendas.
  - Total em aberto (clientes devedores).
  - Vendas por período.
  - Vendas por vendedor (opcional).

---

## 7. Modelo de Dados (Visão Simplificada)

> Obs.: os tipos e tamanhos são apenas exemplo e podem ser ajustados conforme o banco.

### 7.1. Tabela `usuarios`

- `id` (PK, inteiro/UUID)
- `nome` (string)
- `email` (string, único)
- `senha_hash` (string)
- `role` (enum: `ADMIN` | `VENDEDOR`)
- `ativo` (boolean)

### 7.2. Tabela `produtos`

- `id` (PK)
- `nome` (string)
- `descricao` (string opcional)
- `preco` (decimal)
- `ativo` (boolean)

### 7.3. Tabela `clientes`

- `id` (PK)
- `nome` (string)
- `documento` (string opcional – CPF ou outro identificador)
- `telefone` (string opcional)
- `email` (string opcional)
- `ativo` (boolean)

### 7.4. Tabela `vendas`

- `id` (PK)
- `id_cliente` (FK → clientes.id)
- `id_vendedor` (FK → usuarios.id)
- `data_venda` (datetime)
- `valor_total` (decimal)
- `status_pagamento` (enum: `PAGO`, `EM_ABERTO`, `PARCIAL`)

### 7.5. Tabela `itens_venda`

- `id` (PK)
- `id_venda` (FK → vendas.id)
- `id_produto` (FK → produtos.id)
- `quantidade` (int)
- `preco_unitario` (decimal)
- `subtotal` (decimal)

> A partir de `vendas` + `status_pagamento` é possível derivar os dados de “clientes que ainda devem”.

---

## 8. Fluxos Principais

### 8.1. Fluxo — Login

1. Usuário acessa tela de login.
2. Informa usuário e senha.
3. Frontend envia requisição à API de autenticação.
4. Backend valida credenciais.
5. Se ok:
   - Retorna token de sessão (ou mantém sessão no servidor).
   - Informa role (`ADMIN` ou `VENDEDOR`).
6. Frontend redireciona para a tela inicial adequada:
   - Vendedor → tela de vendas/cadastros.
   - Admin → dashboard ou menu administrativo.

### 8.2. Fluxo — Cadastro de Cliente (Vendedor/Admin)

1. Usuário acessa “Cadastrar Cliente”.
2. Preenche dados (nome, documento, contato).
3. Frontend envia para API.
4. Backend valida e salva.
5. Frontend exibe confirmação.

### 8.3. Fluxo — Cadastro de Produto (Vendedor/Admin, de acordo com regra)

1. Usuário acessa “Cadastrar Produto”.
2. Informa nome, preço e detalhes.
3. API valida e grava no banco.
4. Produto fica disponível para uso na tela de vendas.

### 8.4. Fluxo — Registro de Venda

1. Usuário seleciona cliente (ou cadastra um novo).
2. Adiciona um ou mais produtos e quantidades.
3. Sistema calcula valor total.
4. Usuário define status de pagamento (pago ou em aberto).
5. Backend grava `venda` + `itens_venda`.
6. Venda aparece no histórico e impacta dashboards/dívidas.

### 8.5. Fluxo — Consultar Clientes que Devem

1. Usuário acessa tela “Clientes com Dívidas”.
2. Frontend chama endpoint de vendas em aberto.
3. Backend retorna lista de clientes com vendas `EM_ABERTO` e totais.
4. Tela mostra:
   - Nome do cliente.
   - Valor total em aberto.
   - Opcional: link para histórico de vendas pendentes.

### 8.6. Fluxo — Gestão de Vendedores (Admin)

1. Admin acessa tela de “Usuários”.
2. Pode criar usuário com role `VENDEDOR` ou `ADMIN` (conforme política interna).
3. Pode desativar vendedores que saíram da loja.
4. Alterações são refletidas no controle de acesso.

---

## 9. Segurança e Integridade dos Dados

- Armazenar senhas com **hash seguro** (não em texto puro).
- Restringir acesso às rotas de admin no backend (não só no frontend).
- Utilizar chaves estrangeiras para garantir integridade das relações:
  - Vendas sempre associadas a um cliente e a um vendedor.
  - Itens de venda sempre associados a uma venda e a um produto.
- Opcional:
  - Auditoria básica (quem criou/alterou registros e quando).
  - Logs de erro no backend.

---

## 10. Observabilidade e Logs (nível simples)

- Registrar:
  - Erros de aplicação (ex.: falha ao salvar venda, erro de banco).
  - Tentativas de login inválidas.
- Manter logs mínimos para:
  - Depuração de problemas.
  - Melhoria contínua do sistema.

---

## 11. Evolução Futura da Arquitetura

Possíveis extensões:

- Módulo de **estoque** integrado às vendas.
- Relatórios financeiros detalhados (por período, por tipo de produto, por vendedor).
- Integração com sistemas de pagamento ou emissão de nota.
- Controle de múltiplas lojas/unidades.
- Camada de API externa para integrar com outros sistemas.

---
