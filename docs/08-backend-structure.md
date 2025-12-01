# backend-structure
# Estrutura sugerida para o backend Node.js + Express + PostgreSQL

--------------------------------------------------------------------
1. Visão geral
--------------------------------------------------------------------
- Linguagem: JavaScript ou TypeScript
- Framework: Express
- Banco: PostgreSQL (via pg ou knex / Prisma, etc.)
- Autenticação: JWT
- Deploy: Docker + docker-compose

Estrutura de pastas sugerida (dentro de /backend ou /server):

backend/
  package.json
  tsconfig.json (se usar TypeScript)
  Dockerfile
  docker-compose.yml (na raiz do projeto)
  .env (variáveis de ambiente - não commitar)
  src/
    index.ts (ou index.js)
    app.ts (ou app.js)
    config/
      env.ts
      database.ts
    db/
      pool.ts (ou client.ts)       -- conexão com Postgres
    routes/
      auth.routes.ts
      users.routes.ts
      products.routes.ts
      clients.routes.ts
      sales.routes.ts
      debts.routes.ts
      dashboard.routes.ts
    controllers/
      auth.controller.ts
      users.controller.ts
      products.controller.ts
      clients.controller.ts
      sales.controller.ts
      debts.controller.ts
      dashboard.controller.ts
    repositories/ (ou models/)
      users.repository.ts
      products.repository.ts
      clients.repository.ts
      sales.repository.ts
      debts.repository.ts
    middlewares/
      auth.middleware.ts
      error-handler.middleware.ts
      role.middleware.ts
    utils/
      jwt.ts
      password-hash.ts
    types/ (se TypeScript)
      express.d.ts
      domain.d.ts

--------------------------------------------------------------------
2. Fluxo básico de inicialização
--------------------------------------------------------------------
- index.ts
  - Lê variáveis de ambiente
  - Cria instância do app (importado de app.ts)
  - Sobe servidor HTTP na porta configurada

- app.ts
  - Importa Express
  - Configura middlewares globais (JSON, CORS, logger)
  - Configura rotas (importa arquivos de routes)
  - Configura middleware de erro

Exemplo simplificado (TypeScript / JavaScript equivalente):

// src/index.ts
import { app } from './app';
const PORT = process.env.PORT || 3000;

app.listen(PORT, () => {
  console.log(`API rodando na porta ${PORT}`);
});


// src/app.ts
import express from 'express';
import { authRouter } from './routes/auth.routes';
import { usersRouter } from './routes/users.routes';
import { productsRouter } from './routes/products.routes';
import { clientsRouter } from './routes/clients.routes';
import { salesRouter } from './routes/sales.routes';
import { debtsRouter } from './routes/debts.routes';
import { dashboardRouter } from './routes/dashboard.routes';
import { errorHandler } from './middlewares/error-handler.middleware';

export const app = express();

app.use(express.json());

// Rotas públicas
app.use('/api/auth', authRouter);

// Middleware de autenticação (JWT) pode ser aplicado a partir daqui
// app.use(authMiddleware);

// Rotas protegidas
app.use('/api/users', usersRouter);
app.use('/api/products', productsRouter);
app.use('/api/clients', clientsRouter);
app.use('/api/sales', salesRouter);
app.use('/api/debts', debtsRouter);
app.use('/api/dashboard', dashboardRouter);

// Middleware global de tratamento de erros
app.use(errorHandler);


--------------------------------------------------------------------
3. Conexão com o banco (PostgreSQL)
--------------------------------------------------------------------
Exemplo usando pg com pool:

// src/db/pool.ts
import { Pool } from 'pg';

export const pool = new Pool({
  connectionString: process.env.DATABASE_URL,
  // ou usar:
  // user: process.env.DB_USER,
  // host: process.env.DB_HOST,
  // database: process.env.DB_NAME,
  // password: process.env.DB_PASS,
  // port: Number(process.env.DB_PORT) || 5432,
});

export async function query(text: string, params?: any[]) {
  return pool.query(text, params);
}


--------------------------------------------------------------------
4. Middlewares principais
--------------------------------------------------------------------

4.1. auth.middleware.ts (JWT)
- Lê o header Authorization: Bearer <token>
- Valida token JWT
- Anexa dados do usuário em req.user
- Caso inválido, retorna 401

4.2. role.middleware.ts
- Verifica se req.user.perfil está em uma lista de perfis permitidos
- Caso contrário, retorna 403

4.3. error-handler.middleware.ts
- Captura erros lançados pelos controllers
- Normaliza resposta de erro em JSON

--------------------------------------------------------------------
5. Rotas e Controllers (exemplo)
--------------------------------------------------------------------

5.1. Auth

// routes/auth.routes.ts
import { Router } from 'express';
import { login } from '../controllers/auth.controller';

export const authRouter = Router();

authRouter.post('/login', login);

// controllers/auth.controller.ts
import { Request, Response, NextFunction } from 'express';
import { usersRepository } from '../repositories/users.repository';
import { comparePassword } from '../utils/password-hash';
import { signJwt } from '../utils/jwt';

export async function login(req: Request, res: Response, next: NextFunction) {
  try {
    const { usuario, senha } = req.body;

    const user = await usersRepository.findByUsuario(usuario);
    if (!user || !user.ativo) {
      return res.status(401).json({ error: 'Usuário ou senha inválidos' });
    }

    const senhaOk = await comparePassword(senha, user.senha_hash);
    if (!senhaOk) {
      return res.status(401).json({ error: 'Usuário ou senha inválidos' });
    }

    const token = signJwt({
      sub: user.id,
      perfil: user.perfil,
      nome: user.nome,
    });

    return res.json({
      token,
      usuario: {
        id: user.id,
        nome: user.nome,
        usuario: user.usuario,
        perfil: user.perfil,
        ativo: user.ativo,
      },
    });
  } catch (err) {
    next(err);
  }
}

5.2. Exemplo de repository (users)

// repositories/users.repository.ts
import { query } from '../db/pool';

export const usersRepository = {
  async findByUsuario(usuario: string) {
    const result = await query(
      'SELECT * FROM usuarios WHERE usuario = $1 LIMIT 1',
      [usuario]
    );
    return result.rows[0] || null;
  },

  async createUser(data: {
    nome: string;
    usuario: string;
    senha_hash: string;
    perfil: string;
  }) {
    const result = await query(
      `INSERT INTO usuarios (nome, usuario, senha_hash, perfil)
       VALUES ($1, $2, $3, $4)
       RETURNING id, nome, usuario, perfil, ativo`,
      [data.nome, data.usuario, data.senha_hash, data.perfil]
    );
    return result.rows[0];
  },

  // ... outros métodos (listar, atualizar, desativar)
};


--------------------------------------------------------------------
6. Integração com o api-routes
--------------------------------------------------------------------
- O arquivo api-routes.txt (ou api-routes.md) define o CONTRATO da API.
- Cada rota descrita lá deve ter:
  - Uma entrada em /routes/*.routes.ts
  - Um método correspondente em /controllers/*.controller.ts
  - Chamadas a repositories para acessar o banco.

Sugestão:
- Manter os nomes alinhados:
  - GET /api/products -> products.routes.ts -> products.controller.list
  - POST /api/products -> products.controller.create
  - etc.

--------------------------------------------------------------------
7. Docker (resumo)
--------------------------------------------------------------------
- Dockerfile (backend):
  - Usa node:XX-alpine
  - Copia package.json / yarn.lock
  - Instala dependências
  - Copia src/
  - Roda build (se TypeScript) e comando de start

- docker-compose.yml:
  - service "backend" (Node)
  - service "db" (Postgres)
  - Volume para dados do Postgres
  - Rede compartilhada entre backend e db

--------------------------------------------------------------------
8. Próximos passos
--------------------------------------------------------------------
- Implementar autenticação JWT completa.
- Implementar rotas CRUD conforme api-routes.txt.
- Implementar consultas específicas (clientes devedores, dashboards).
- Adicionar testes (unitários/integrados) se desejar.
