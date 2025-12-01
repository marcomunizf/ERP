# SPEC – Sistema ERP de Controle de Loja

> Versão: 0.1  
> Autor: [Marco Muniz]  
> Data: [01/12/2025]  
> Status:  Rascunho / Em uso / Aprovado
              X

---

## 1. Visão Geral

### 1.1. Contexto

O sistema de controle de loja tem como objetivo centralizar o cadastro de produtos, clientes e vendas, permitindo acompanhar o faturamento de forma simples e confiável.

O sistema será utilizado em um ambiente de pequena/média loja, com foco em:

- Cadastro e consulta rápida de produtos
- Registro de vendas
- Acompanhamento básico de faturamento
- Registro de clientes para histórico de compras

### 1.2. Objetivos

- **O1:** Reduzir o uso de planilhas soltas / controles manuais.  
- **O2:** Facilitar o cadastro e a consulta de produtos e clientes.  
- **O3:** Permitir o registro rápido de vendas no caixa.  
- **O4:** Oferecer visão de faturamento diário, semanal e mensal.  
- **O5:** Preparar uma base de dados limpa para futuras análises (BI, relatórios, etc.).

### 1.3. Escopo (MVP)

Incluído no MVP:

- Autenticação básica (login/logout)
- Cadastro de Produtos
- Cadastro de Clientes
- Registro de Vendas (com itens)
- Visão de Faturamento (dashboard simples)
- Consulta / filtros básicos

Fora do escopo do MVP (pode entrar em versões futuras):

- Controle de estoque avançado (entrada/saída, inventário completo)

---

## 2. Perfis de Usuário

### 2.1. Usuário Administrador

- Pode criar/editar/remover usuários do sistema.
- Pode criar/remover produtos.
- Pode criar/remover clientes.  
- Tem acesso a todos os relatórios e cadastros. 

### 2.2. Usuário Vendedor / Operador de Caixa

- Pode logar no sistema.
- Pode criar produtos.
- Pode criar clientes.
- Pode registrar vendas.  
- Pode consultar produtos e clientes.  
- Não pode excluir usuários ou alterar configurações críticas.

---

## 3. Glossário

- **Produto:** Item físico ou serviço vendido pela loja.  
- **Cliente:** Pessoa física ou jurídica que realiza compras.  
- **Venda:** Transação composta por um cabeçalho (data, cliente, total) e itens (produto, quantidade, preço).  
- **Item de Venda:** Linha da venda, associando um produto a quantidade e valor.  
- **Faturamento:** Soma dos valores de venda em determinado período.  

---

## 4. Modelo de Domínio (conceitual)

### 4.1. Entidades Principais

- **Produto**
  - id
  - nome
  - descricao
  - preco_unitario
  - codigo_barras (opcional)
  - categoria (opcional)
  - ativo (boolean)

- **Cliente**
  - id
  - nome
  - documento (CPF/CNPJ) [opcional no MVP]
  - telefone
  - email [opcional]
  - observacoes [opcional]
  - ativo (boolean)

- **Usuario**
  - id
  - nome
  - email
  - senha_hash
  - perfil (ADMIN / VENDEDOR)
  - ativo (boolean)

- **Venda**
  - id
  - data_hora
  - id_cliente [opcional – venda pode ser “cliente não cadastrado”]
  - id_usuario_vendedor
  - valor_total
  - forma_pagamento (DINHEIRO / CARTAO / PIX / OUTRO)
  - observacoes [opcional]

- **ItemVenda**
  - id
  - id_venda
  - id_produto
  - quantidade
  - preco_unitario
  - desconto_valor [opcional]
  - subtotal

---

## 5. Requisitos Funcionais

> Estrutura sugerida: ID, Nome, Descrição, Fluxo principal, Regras de negócio, Critérios de aceite.

### RF-01 – Autenticação de Usuário

**Descrição:**  
Permitir que usuários cadastrados façam login no sistema com usuário e senha.

**Fluxo principal:**
1. Usuário acessa a tela de login.
2. Informa usuário e senha.
3. Sistema valida credenciais.
4. Em caso de sucesso, redireciona para o dashboard inicial.
5. Em caso de erro, exibe mensagem genérica: “Usuário ou senha inválidos”.

**Regras de negócio:**
- Senha deve ser armazenada com hash (nunca em texto puro).
- Usuários inativos não podem acessar o sistema.

**Critérios de aceite:**
- CA1: Usuário com usuário/senha corretos acessa a área logada.
- CA2: Usuário com usuário ou senha incorretos recebe mensagem e permanece na tela de login.
- CA3: Usuário inativo não consegue fazer login.

---

### RF-02 – Cadastro de Produtos

**Descrição:**  
Permitir criar, editar, listar e inativar produtos.

**Fluxo principal (criação):**
1. Usuário acessa a tela de produtos.
2. Clica em “Novo Produto”.
3. Informa: nome, preço unitário e (opcionalmente) descrição, código de barras, categoria.
4. Salva o produto.
5. Sistema confirma criação.

**Regras de negócio:**
- Campo `nome` é obrigatório.
- Campo `preco_unitario` é obrigatório e deve ser maior ou igual a 0.
- Exclusão física de produtos não deve acontecer no MVP: usar flag `ativo=false` ao invés de deletar.
- Produtos inativos não devem aparecer na pesquisa padrão (usar filtro “mostrar inativos” opcional).

**Critérios de aceite:**
- CA1: É possível criar um produto com campos obrigatórios.
- CA2: É possível editar um produto existente.
- CA3: É possível inativar um produto, e este deixa de aparecer na listagem padrão.
- CA4: Produtos inativos podem ser visualizados se o usuário marcar “mostrar inativos”.

---

### RF-03 – Cadastro de Clientes

**Descrição:**  
Permitir criar, editar, listar e inativar clientes.

**Fluxo principal (criação):**
1. Usuário acessa a tela de clientes.
2. Clica em “Novo Cliente”.
3. Informa ao menos o nome do cliente.
4. (Opcional) Preenche documento, telefone, email.
5. Salva o registro.

**Regras de negócio:**
- Campo `nome` é obrigatório.
- Não é obrigatório ter documento no MVP.
- Exclusão física também deve ser evitada; usar `ativo=false`.

**Critérios de aceite:**
- CA1: É possível cadastrar cliente apenas com nome.
- CA2: Cliente inativo não aparece em buscas padrão.
- CA3: Cliente inativo não deve ser selecionável como cliente de uma nova venda.

---

### RF-04 – Registro de Venda

**Descrição:**  
Permitir que o usuário registre uma venda com um ou mais itens.

**Fluxo principal:**
1. Usuário acessa a tela de “Nova Venda”.
2. (Opcional) Seleciona um cliente; se não selecionar, venda é registrada como “Cliente não identificado”.
3. Adiciona itens:
   - Pesquisa produto por nome ou código de barras.
   - Informa quantidade.
   - Sistema sugere o preço unitário do cadastro do produto (pode permitir override ou não, definido na regra de negócio).
4. Sistema calcula subtotal de cada item e total da venda.
5. Usuário seleciona forma de pagamento.
6. Usuário confirma a venda.
7. Sistema registra a venda e exibe um resumo.

**Regras de negócio:**
- Uma venda deve ter pelo menos 1 item.
- Quantidade deve ser > 0.
- Preço unitário ≥ 0.
- O valor total da venda é a soma dos subtotais dos itens.
- Data/hora da venda deve ser salva no momento da confirmação.

**Critérios de aceite:**
- CA1: Não é possível confirmar venda sem itens.
- CA2: Não é possível adicionar item com quantidade ≤ 0.
- CA3: O total exibido deve ser exatamente a soma dos itens.
- CA4: Ao confirmar, a venda fica registrada e pode ser consultada em “Histórico de vendas”.

---

### RF-05 – Consulta de Vendas / Histórico

**Descrição:**  
Permitir filtrar e visualizar vendas por período, cliente e forma de pagamento.

**Filtros mínimos:**
- Data inicial / Data final
- Cliente (opcional)
- Forma de pagamento (opcional)

**Informações na listagem:**
- Data/hora
- Cliente (ou “Não identificado”)
- Valor total
- Forma de pagamento

**Critérios de aceite:**
- CA1: Usuário consegue listar vendas de um período.
- CA2: Usuário consegue filtrar por cliente.
- CA3: Ao clicar em uma venda, consegue visualizar os itens.

---

### RF-06 – Dashboard de Faturamento

**Descrição:**  
Exibir visão resumida do faturamento em um período.

**Métricas mínimas:**
- Total faturado no dia atual.
- Total faturado no mês atual.
- Gráfico simples (barra ou linha) com faturamento dos últimos X dias (ex.: 7 ou 30 dias).

**Critérios de aceite:**
- CA1: Valores apresentados batem com a soma das vendas do período.
- CA2: Usuário pode mudar o período (dia/semana/mês) e ver os dados atualizados.

---

## 6. Requisitos Não Funcionais (RNF)

### RNF-01 – Segurança

- Senhas armazenadas com hash seguro.
- Não exibir dados sensíveis em logs.
- Implementar controle básico de sessão (logout, expiração de sessão após 120 minutos de inatividade).

### RNF-02 – Desempenho

- Listagem de produtos e vendas deve carregar em até ~2 segundos para conjuntos de dados típicos (até alguns milhares de registros).

---

## 7. Modelo de Dados (rascunho de tabelas)

> Este é um modelo inicial para guiar a geração de código / migrations.

### Tabela `usuarios`

- id (PK)
- nome (string)
- email (string, único)
- senha_hash (string)
- perfil (enum: ADMIN, VENDEDOR)
- ativo (boolean, default true)
- created_at (datetime)
- updated_at (datetime)

### Tabela `clientes`

- id (PK)
- nome (string)
- documento (string, opcional)
- telefone (string, opcional)
- email (string, opcional)
- observacoes (text, opcional)
- ativo (boolean, default true)
- created_at (datetime)
- updated_at (datetime)

### Tabela `produtos`

- id (PK)
- nome (string)
- descricao (text, opcional)
- preco_unitario (numeric)
- codigo_barras (string, opcional, único se usado)
- categoria (string, opcional)
- ativo (boolean, default true)
- created_at (datetime)
- updated_at (datetime)

### Tabela `vendas`

- id (PK)
- data_hora (datetime)
- id_cliente (FK clientes.id, opcional)
- id_usuario_vendedor (FK usuarios.id)
- valor_total (numeric)
- forma_pagamento (string ou enum)
- observacoes (text, opcional)
- created_at (datetime)
- updated_at (datetime)

### Tabela `itens_venda`

- id (PK)
- id_venda (FK vendas.id)
- id_produto (FK produtos.id)
- quantidade (numeric)
- preco_unitario (numeric)
- desconto_valor (numeric, default 0)
- subtotal (numeric)
- created_at (datetime)
- updated_at (datetime)

---

## 8. Casos de Teste (aceitação)

### CT-01 – Login com credenciais válidas

**Dado** um usuário ativo com usuario e senha cadastrados  
**Quando** ele informar usuario e senha corretos  
**Então** deve acessar o dashboard inicial.

### CT-02 – Cadastro de produto sem nome

**Quando** o usuário tentar salvar um produto sem preencher o nome  
**Então** o sistema deve exibir mensagem de erro e não salvar o registro.

### CT-03 – Registro de venda com um item

**Dado** pelo menos um produto cadastrado  
**Quando**
- Usuário abrir “Nova Venda”
- Adicionar um item com quantidade > 0
- Confirmar a venda  
**Então** a venda deve ser criada, com total igual ao subtotal do item.

### CT-04 – Dashboard de faturamento diário

**Dado** vendas registradas para a data de hoje  
**Quando** o usuário acessar o dashboard  
**Então** o total faturado do dia deve ser igual à soma das vendas de hoje.

---

## 9. Instruções:
> Não invente entidades além das descritas na SPEC.

---
