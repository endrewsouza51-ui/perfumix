# Perfumix — Loja Online (Frontend + Admin + Backend API)

Perfumix é uma loja online estática (Netlify) com API em Node.js/Express e painel administrativo simples. O pacote está pronto para uso com placeholders de pagamentos e banco de dados (mock em memória) e exemplos para Postgres/Prisma.

## Estrutura

- `frontend/`
  - Site público estático: `index.html`, `store.html`, `product.html`, `cart.html`, `checkout.html`, `account.html`, `faq.html`, `privacy.html`, `terms.html`
  - Estilos: `css/style.css`
  - Scripts: `js/app.js`
  - Assets: `assets/`
- `admin/`
  - Painel: `index.html`, `css/style.css`, `js/app.js`
- `backend/`
  - `package.json`, `Dockerfile`
  - `src/`
    - `server.js`
    - `middleware/auth.js` (JWT + role)
    - `store/db.js` (mock em memória)
    - `routes/`
      - `auth.js` (register/login/forgot)
      - `products.js` (CRUD)
      - `orders.js` (listar, alterar status)
      - `checkout.js` (criar pedido)
      - `payments.js` (placeholder criar transação)
      - `settings.js` (configurações do site/pagamentos)
      - `customers.js` (lista de clientes)
  - `prisma/schema.prisma` (exemplo)
- `docker-compose.yml` (api + postgres + nginx)
- `nginx.conf` (proxy reverso para API)
- `env.example` (variáveis de ambiente)

## UI e Identidade

- Tema: preto `#000000` e vermelho `#E10600`.
- Layout responsivo, cartões de produto, microinterações, toasts.
- Acessibilidade básica: labels e contraste adequado.

## Funcionalidades do Frontend

- Catálogo com filtros por categoria, busca e ordenação.
- Página de produto com galeria, preço, descrição e avaliações simuladas.
- Carrinho com adicionar/editar/remover, subtotal, frete (fixo/grátis > X), descontos por cupom (simulado), impostos, total e atualização dinâmica.
- Checkout com validações (CPF/CEP/email) e envio do payload para `POST /api/payments/create` (placeholder).
- Minha conta: registro/login, recuperação de senha (simulada) e listagem de pedidos via `GET /api/orders` (autenticado).
- SEO básico: metas e OG tags.

## Painel Admin

- Login admin (usuario seed: `admin@perfumix.local` / senha `admin123`).
- Dashboard com KPIs (pedidos, receita, produtos).
- Produtos: CRUD com modal, export CSV.
- Pedidos: listagem e alteração de status (Pendente → Processando → Enviado → Concluído → Cancelado), export CSV.
- Clientes: listagem.
- Pagamentos: formulário para ativar/desativar Pix/Cartão/Boleto e inserir chaves (placeholders).
- Editor visual: alterar cor primária, textos de banner e links do menu (preview e publicar em `site-settings`).
- Logs simples em memória.

## Backend API (Node.js/Express)

- Auth: `POST /api/auth/register`, `POST /api/auth/login`, `POST /api/auth/forgot-password`.
- Products: `GET /api/products`, `GET /api/products/:id`, `POST /api/products`, `PATCH /api/products/:id`, `DELETE /api/products/:id`.
- Orders: `GET /api/orders` (com `?summary=1`), `GET /api/orders/:id`, `PATCH /api/orders/:id/status`.
- Checkout: `POST /api/checkout` (cria pedido em memória).
- Payments: `POST /api/payments/create` (placeholder), `POST /api/payments/webhook` (exemplo).
- Site settings: `GET /api/site-settings`, `PUT /api/site-settings`.
- JWT + role guard em `middleware/auth.js`.

### Integração com Gateway de Pagamento (placeholders)

- Inserir credenciais no backend via env (`PAYMENT_PIX_KEY`, `PAYMENT_CARD_KEY`, `PAYMENT_BOLETO_KEY`).
- Alternar ambiente com `PAYMENT_ENV=sandbox|production`.
- O front envia o pedido para `/api/payments/create`. A implementação real deve:
  - Criar a transação no gateway.
  - Retornar `payment_url` para redirecionamento ou instruções.
- Comentários no código: "AQUI INSERIR CREDENCIAIS DO GATEWAY".

### Banco de Dados

- Mock em memória por padrão (`backend/src/store/db.js`).
- Para usar Postgres/Prisma:
  - Ajustar `backend/prisma/schema.prisma` (provider + models).
  - Definir `DB_URL` para Postgres (ex.: no Compose) ou SQLite para dev.
  - Instalar Prisma: `npm i -D prisma && npm i @prisma/client`.
  - Rodar `npx prisma generate` e `npx prisma migrate dev`.
  - Substituir operações de `db.js` por chamadas ao ORM.

## Variáveis de Ambiente

Veja `env.example` e crie `.env` na raiz.

- `PORT`: porta do backend (default 4000)
- `JWT_SECRET`: segredo do JWT (troque em produção)
- `DB_URL`: URL do banco (Postgres/SQLite)
- `PAYMENT_ENV`: sandbox|production
- `PAYMENT_PIX_KEY`, `PAYMENT_CARD_KEY`, `PAYMENT_BOLETO_KEY`
- `MOCK_PAYMENT_REDIRECT`: URL de redirecionamento em modo simulado

## Scripts NPM

- Backend (`backend/package.json`):
  - `npm run dev` — inicia API com watch
  - `npm start` — inicia API
- Frontend (`frontend/package.json`):
  - `npm run dev` — serve estático local em `http://localhost:5173`
  - `npm start` — idem
  - `npm run build` — no-op (site já é estático)

## Rodando Localmente (sem Docker)

1. Backend
   - `cd backend`
   - `npm install`
   - `cp ../env.example ../.env` (ou crie sua `.env`)
   - `npm run dev`
   - API em `http://localhost:4000`

2. Frontend e Admin
   - Abra os HTML diretamente ou sirva:
   - `cd frontend && npm install && npm run dev` (opcional para desenvolvimento)
   - Configure a URL do backend no navegador: `localStorage.setItem('PERFUMIX_API','http://localhost:4000')`
   - Acesse `frontend/index.html` e `admin/index.html`

## Deploy — Frontend (Netlify)

1. Faça push do repositório para GitHub/GitLab.
2. No Netlify, crie um novo site a partir do repositório.
3. Configure:
   - Build command: vazio (site estático)
   - Publish directory: `frontend`
4. (Opcional) Variáveis de ambiente no Netlify para apontar a API:
   - `PERFUMIX_API` (ou configure via localStorage no browser).
5. Após o deploy, teste: `https://seusite.netlify.app`

## Deploy — Backend (Docker Compose em Ubuntu)

1. Instale Docker e Docker Compose.
2. Copie os arquivos do projeto para o servidor.
3. Crie `.env` na raiz a partir de `env.example`.
4. Execute:
   ```bash
   docker compose up -d --build
   ```
5. A API estará em `http://<host>:4000` (direto) e via Nginx em `http://<host>:8080/api/`.
6. Segurança:
   - Configure HTTPS (Nginx com TLS via certbot/Load Balancer)
   - Defina `JWT_SECRET` forte
   - Restrinja portas públicas conforme necessário

## Integração de Pagamentos – Onde inserir credenciais

- Backend `payments.js`:
  - Use `process.env.PAYMENT_*`.
  - Troque a chamada mock por SDK/HTTP do gateway:
    ```js
    // Exemplo (pseudo):
    const client = new Gateway({ key: process.env.PAYMENT_CARD_KEY, env: process.env.PAYMENT_ENV });
    const tx = await client.payments.create({ amount: totals.total, currency: 'BRL', ... });
    return res.status(201).json({ id: tx.id, payment_url: tx.checkoutUrl });
    ```
- Habilite/Desabilite métodos em `site-settings` via painel admin.

## Observações

- Comentários no código indicam claramente:
  - "AQUI INSERIR CREDENCIAIS DO GATEWAY"
  - "AQUI CONFIGURAR URL DO BACKEND"
- HTTPS/SSL é obrigatório para pagamentos em produção.
- Sanitização e validação básica aplicadas (frontend e backend). Para produção, aumente o rigor e adicione rate limiting, CORS strict e logs persistentes.

## Produtos Iniciais

- 6 produtos de exemplo já presentes no front e no backend em memória.

---

Feito com ❤️ para um lançamento rápido. Insira suas chaves de pagamento, aponte o frontend para a API e publique.
