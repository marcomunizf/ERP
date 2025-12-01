# database-schema
# Esquema do banco de dados PostgreSQL para o Sistema ERP da Loja

-- ------------------------------------------------------------------
-- 1. Informações gerais
-- ------------------------------------------------------------------
-- Banco: PostgreSQL
-- Objetivo: armazenar usuários, produtos, clientes, vendas e itens de venda,
--           permitindo controle de dívidas (vendas EM_ABERTO).

-- OBS: Ajuste nomes de schema (public) ou prefixos conforme necessário.


-- ------------------------------------------------------------------
-- 2. Tabela: usuarios
-- ------------------------------------------------------------------
CREATE TABLE IF NOT EXISTS usuarios (
    id              SERIAL PRIMARY KEY,
    nome            VARCHAR(150) NOT NULL,
    usuario         VARCHAR(50)  NOT NULL UNIQUE,   -- login
    senha_hash      VARCHAR(255) NOT NULL,
    perfil          VARCHAR(20)  NOT NULL CHECK (perfil IN ('ADMIN', 'VENDEDOR')),
    ativo           BOOLEAN      NOT NULL DEFAULT TRUE,
    created_at      TIMESTAMP    NOT NULL DEFAULT NOW(),
    updated_at      TIMESTAMP    NOT NULL DEFAULT NOW()
);

CREATE INDEX IF NOT EXISTS idx_usuarios_perfil
    ON usuarios (perfil);

CREATE INDEX IF NOT EXISTS idx_usuarios_ativo
    ON usuarios (ativo);


-- ------------------------------------------------------------------
-- 3. Tabela: produtos
-- ------------------------------------------------------------------
CREATE TABLE IF NOT EXISTS produtos (
    id              SERIAL PRIMARY KEY,
    nome            VARCHAR(150) NOT NULL,
    descricao       TEXT,
    preco_unitario  NUMERIC(12,2) NOT NULL CHECK (preco_unitario >= 0),
    codigo_barras   VARCHAR(50),
    categoria       VARCHAR(100),
    ativo           BOOLEAN      NOT NULL DEFAULT TRUE,
    created_at      TIMESTAMP    NOT NULL DEFAULT NOW(),
    updated_at      TIMESTAMP    NOT NULL DEFAULT NOW()
);

CREATE UNIQUE INDEX IF NOT EXISTS idx_produtos_codigo_barras
    ON produtos (codigo_barras)
    WHERE codigo_barras IS NOT NULL;

CREATE INDEX IF NOT EXISTS idx_produtos_nome
    ON produtos (nome);

CREATE INDEX IF NOT EXISTS idx_produtos_ativo
    ON produtos (ativo);


-- ------------------------------------------------------------------
-- 4. Tabela: clientes
-- ------------------------------------------------------------------
CREATE TABLE IF NOT EXISTS clientes (
    id              SERIAL PRIMARY KEY,
    nome            VARCHAR(150) NOT NULL,
    documento       VARCHAR(30),          -- CPF/CNPJ ou outro
    telefone        VARCHAR(30),
    email           VARCHAR(150),
    observacoes     TEXT,
    ativo           BOOLEAN      NOT NULL DEFAULT TRUE,
    created_at      TIMESTAMP    NOT NULL DEFAULT NOW(),
    updated_at      TIMESTAMP    NOT NULL DEFAULT NOW()
);

CREATE INDEX IF NOT EXISTS idx_clientes_nome
    ON clientes (nome);

CREATE INDEX IF NOT EXISTS idx_clientes_ativo
    ON clientes (ativo);


-- ------------------------------------------------------------------
-- 5. Tabela: vendas
-- ------------------------------------------------------------------
CREATE TABLE IF NOT EXISTS vendas (
    id                  SERIAL PRIMARY KEY,
    data_hora           TIMESTAMP    NOT NULL DEFAULT NOW(),
    id_cliente          INTEGER      REFERENCES clientes(id),
    id_usuario_vendedor INTEGER      NOT NULL REFERENCES usuarios(id),
    valor_total         NUMERIC(12,2) NOT NULL DEFAULT 0,
    status_pagamento    VARCHAR(20)  NOT NULL CHECK (status_pagamento IN ('PAGO', 'EM_ABERTO')),
    forma_pagamento     VARCHAR(20)  NOT NULL DEFAULT 'OUTRO',
    observacoes         TEXT,
    created_at          TIMESTAMP    NOT NULL DEFAULT NOW(),
    updated_at          TIMESTAMP    NOT NULL DEFAULT NOW()
);

CREATE INDEX IF NOT EXISTS idx_vendas_data_hora
    ON vendas (data_hora);

CREATE INDEX IF NOT EXISTS idx_vendas_id_cliente
    ON vendas (id_cliente);

CREATE INDEX IF NOT EXISTS idx_vendas_id_usuario_vendedor
    ON vendas (id_usuario_vendedor);

CREATE INDEX IF NOT EXISTS idx_vendas_status_pagamento
    ON vendas (status_pagamento);


-- ------------------------------------------------------------------
-- 6. Tabela: itens_venda
-- ------------------------------------------------------------------
CREATE TABLE IF NOT EXISTS itens_venda (
    id              SERIAL PRIMARY KEY,
    id_venda        INTEGER NOT NULL REFERENCES vendas(id) ON DELETE CASCADE,
    id_produto      INTEGER NOT NULL REFERENCES produtos(id),
    quantidade      NUMERIC(12,3) NOT NULL CHECK (quantidade > 0),
    preco_unitario  NUMERIC(12,2) NOT NULL CHECK (preco_unitario >= 0),
    desconto_valor  NUMERIC(12,2) NOT NULL DEFAULT 0,
    subtotal        NUMERIC(12,2) NOT NULL,
    created_at      TIMESTAMP     NOT NULL DEFAULT NOW(),
    updated_at      TIMESTAMP     NOT NULL DEFAULT NOW()
);

CREATE INDEX IF NOT EXISTS idx_itens_venda_id_venda
    ON itens_venda (id_venda);

CREATE INDEX IF NOT EXISTS idx_itens_venda_id_produto
    ON itens_venda (id_produto);


-- ------------------------------------------------------------------
-- 7. Visões auxiliares (opcional)
-- ------------------------------------------------------------------
-- Visão: clientes_devedores
-- Mostra clientes com valor em aberto (soma de vendas EM_ABERTO).

CREATE OR REPLACE VIEW vw_clientes_devedores AS
SELECT
    c.id              AS id_cliente,
    c.nome            AS nome_cliente,
    SUM(v.valor_total) AS total_em_aberto,
    COUNT(v.id)       AS quantidade_vendas_em_aberto
FROM clientes c
JOIN vendas v ON v.id_cliente = c.id
WHERE v.status_pagamento = 'EM_ABERTO'
GROUP BY c.id, c.nome
HAVING SUM(v.valor_total) > 0;


-- ------------------------------------------------------------------
-- 8. Observações
-- ------------------------------------------------------------------
-- 1) Os campos created_at / updated_at podem ser atualizados via aplicação
--    ou via triggers (não definidos aqui).
-- 2) Ajustar tamanhos de VARCHAR conforme necessidade real.
-- 3) Caso queira schemas separados, prefixar como: loja.usuarios, etc.

