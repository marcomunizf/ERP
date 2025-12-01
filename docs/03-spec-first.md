# SPEC – Sistema ERP de Controle de Loja
> Versão: 0.2  
> Autor: Marco Muniz  
> Data: 01/12/2025  
> Status: Rascunho

---

# 1. Visão Geral

## 1.1. Contexto

O sistema ERP interno da loja centraliza cadastros de produtos, clientes, vendas e dívidas.  
É utilizado exclusivamente por **Admins** e **Vendedores**, com controle de acesso por **login (usuário + senha)**.

## 1.2. Objetivos

- **O1:** Reduzir uso de controles manuais e planilhas.
- **O2:** Permitir cadastro ágil de produtos e clientes.
- **O3:** Registrar vendas com rapidez e precisão.
- **O4:** Oferecer visão clara de clientes que devem.
- **O5:** Fornecer dashboards administrativos para tomada de decisão.
- **O6:** Criar uma base sólida para análises e expansão futura.

## 1.3. Escopo do MVP

Incluído:

- Login por usuário + senha
- Gestão de usuários (apenas admin)
- Cadastro de produtos
- Cadastro de clientes
- Registro de vendas
- Controle de dívidas (vendas em aberto)
- Listagem de clientes devedores
- Dashboard simples para Admin

Fora do MVP:

- Controle de estoque avançado
- Integração com pagamentos externos
- Multi-loja
- App mobile

---

# 2. Perfis de Usuário

## 2.1. Usuário Administrador (ADMIN)

- Pode criar/editar/desativar usuários (vendedores e admins).
- Pode criar, editar e inativar produtos.
- Pode criar, editar e inativar clientes.
- Pode registrar vendas.
- Acessa dashboards exclusivos.
- Acessa relatórios e visão consolidada de dívidas.

## 2.2. Usuário Vendedor (VENDEDOR)

- Pode logar no sistema.
- Pode cadastrar clientes.
- Pode registrar vendas.
- Pode consultar produtos e clientes.
- Pode ver lista de clientes devedores.
- **Não pode**: criar usuários, excluir dados, acessar dashboards administrativos, criar produtos.

---

# 3. Glossário

- **Usuário:** pessoa autorizada a acessar o sistema.
- **Produto:** item comercializado pela loja.
- **Cliente:** pessoa que compra e pode ter saldo em aberto.
- **Venda:** conjunto de itens vendidos para um cliente.
- **Item de Venda:** produto com quantidade + preço.
- **Dívida:** venda cujo status de pagamento está “EM_ABERTO”.
- **Dashboard:** visão visual agregada com KPIs.

---

# 4. Modelo de Domínio (Conceitual)

## 4.1. Entidades Principais

### Usuário
- id
- nome
- usuario (login) — string única
- senha_hash
- perfil (ADMIN / VENDEDOR)
- ativo (boolean)

### Produto
- id
- nome
- descricao (opcional)
- preco_unitario
- codigo_barras (opcional)
- categoria (opcional)
- ativo (boolean)

### Cliente
- id
- nome
- documento (opcional)
- telefone (opcional)
- email (opcional)
- observacoes (opcional)
- ativo (boolean)

### Venda
- id
- data_hora
- id_cliente (opcional)
- id_usuario_vendedor
- valor_total
- status_pagamento (PAGO / EM_ABERTO)
- forma_pagamento (DINHEIRO / CARTAO / PIX / OUTRO)
- observacoes (opcional)

### ItemVenda
- id
- id_venda
- id_produto
- quantidade
- preco_unitario
- desconto_valor (opcional)
- subtotal

---

# 5. Requisitos Funcionais

> Estrutura: descrição, fluxo principal, regras de negócio, critérios de aceite.

---

## RF-01 — Autenticação por Usuário e Senha

**Descrição:**  
Permitir login com *usuário + senha*, validando perfis e permissões.

**Fluxo:**
1. Usuário entra na tela de login.
2. Informa usuário e senha.
3. Sistema valida credenciais.
4. Em sucesso: redireciona para tela inicial do perfil.
5. Em falha: exibe “Usuário ou senha inválidos”.

**Regras:**
- Senha armazenada com hash.
- Usuário inativo não acessa.

**Critérios de aceite:**
- CA1: Login válido acessa área logada.
- CA2: Login inválido exibe erro.
- CA3: Usuário desativado não acessa.

---

## RF-02 — Gestão de Usuários (Admin)

**Descrição:**  
Permitir que admins gerenciem vendedores e outros admins.

**Fluxo:**
1. Admin acessa tela “Usuários”.
2. Pode criar, editar, desativar usuários.
3. Sistema garante unicidade do campo `usuario`.

**Regras:**
- Exclusão física não permitida (usar ativo=false).
- Perfil define permissões.

**Critérios de aceite:**
- CA1: Admin cria usuário com sucesso.
- CA2: Usuário desativado não consegue logar.
- CA3: Sistema impede criar dois usuários com mesmo login.

---

## RF-03 — Cadastro de Produtos (Admin)

**Descrição:**  
Admin registra produtos com nome e preço.

**Fluxo:**
1. Admin acessa “Produtos”.
2. Clica em “Novo”.
3. Preenche campos obrigatórios.
4. Salva.

**Regras:**
- Nome obrigatório.
- Preço >= 0.
- Inativação ao invés de exclusão.

**Critérios de aceite:**
- Produto criado, editado e inativado corretamente.
- Produto inativo não aparece em vendas.

---

## RF-04 — Cadastro de Clientes (Admin e Vendedor)

**Descrição:**  
Cadastrar clientes para registro de vendas e histórico.

**Fluxo:**
1. Acessar “Clientes”.
2. Clicar “Novo Cliente”.
3. Preencher nome.
4. Salvar.

**Regras:**
- Nome obrigatório.
- Inativação sem exclusão física.

**Critérios de aceite:**
- Cliente criado com sucesso.
- Inativos não aparecem em vendas.

---

## RF-05 — Registro de Vendas

**Descrição:**  
Registrar venda de um ou mais produtos para um cliente.

**Fluxo:**
1. Usuário acessa “Nova Venda”.
2. Seleciona cliente (ou marca como “não cadastrado”).
3. Adiciona itens.
4. Sistema calcula totais.
5. Define status de pagamento.
6. Confirma.

**Regras:**
- Produtos inativos não aparecem.
- Subtotal = quantidade × preço.
- Total = soma dos itens – descontos.

**Critérios de aceite:**
- Venda registrada com itens corretos.
- Total calculado automaticamente.
- Venda “EM_ABERTO” aparece como dívida.

---

## RF-06 — Gestão de Dívidas / Clientes Devedores

**Descrição:**  
Exibir clientes com vendas em aberto e o valor total devido.

**Fluxo:**
1. Usuário acessa “Clientes que Devem”.
2. Sistema lista clientes com vendas EM_ABERTO.
3. Exibe total em aberto por cliente.

**Regras:**
- Somar todas as vendas pendentes.
- Considera status_pagamento.

**Critérios de aceite:**
- Lista correta mesmo com múltiplas vendas abertas.
- Clientes sem dívidas não aparecem.

---

## RF-07 — Dashboard Administrativo (Admin)

**Descrição:**  
Admin acessa dashboard com indicadores.

**Indicadores:**
- Faturamento do dia.
- Faturamento do mês.
- Total em aberto (dívidas).
- Quantidade de vendas no período.
- Vendas por vendedor (opcional).

**Critérios de aceite:**
- Valores e somas devem refletir o banco.
- Admin exclusivo.

---

# 6. Requisitos Não Funcionais

## RNF-01 — Usabilidade
- Fluxos simples, poucos cliques para registrar venda.

## RNF-02 — Performance
- Listagem de produtos/ clientes deve carregar em até 2s com volume inicial.

## RNF-03 — Segurança
- Hash de senha.
- Perfis protegidos por backend (não só no frontend).
- Usuários inativos não acessam.

## RNF-04 — Persistência
- Banco PostgreSQL.
- Dados persistidos via volume Docker.

## RNF-05 — Backup
- Backup diário (definido pela loja).

## RNF-06 — Disponibilidade
- Disponível durante funcionamento da loja.

---

# 7. Casos de Teste (Aceitação)

## CT-01 — Login válido
**Dado** usuário ativo  
**Quando** informar usuário + senha corretos  
**Então** deve acessar sistema.

## CT-02 — Login inválido
**Quando** informar senha incorreta  
**Então** exibir mensagem de erro.

## CT-03 — Cadastro de Produto sem nome
Erro ao tentar salvar.

## CT-04 — Registro de Venda
Venda com item único deve calcular total automaticamente.

## CT-05 — Cliente Devedor
Cliente com duas vendas EM_ABERTO deve aparecer com soma total correta.

## CT-06 — Dashboard Admin
Faturamento diário deve refletir soma real das vendas do dia.

---

# 8. Instruções

- Não criar entidades fora desta SPEC.
- Alterações estruturais devem ser documentadas.
