# xiaozhi-mcphub-hdkp: Personal fork of xiaozhi-mcphub

[中文版](README.zh.md) | English | [Tiếng Việt](README.vi.md)

> **Forked from** [huangjunsen0406/xiaozhi-mcphub](https://github.com/huangjunsen0406/xiaozhi-mcphub) by Huang Junsen.
> Vietnamese localization reference: [hdkp81/xiaozhi-mcphub-vn](https://github.com/hdkp81/xiaozhi-mcphub-vn).
> This fork keeps the original `LICENSE` (Apache License 2.0) and `NOTICE`.

**xiaozhi-mcphub-hdkp** is my personal version of xiaozhi-mcphub, an MCP (Model Context Protocol) bridge and management console optimized for the Xiaozhi AI platform. It provides multi-endpoint management, automatic reconnection, vector-based smart routing, and OpenAPI-compatible access.

![Dashboard Preview](assets/dashboard.png)

## 🔄 Changes in This Fork

Compared with the upstream project, this fork:

- Builds the Docker image from source (`build: .`) instead of pulling a prebuilt image.
- Requires `JWT_SECRET` to be set in `.env` (Docker Compose stops with a clear message if it is missing).
- Supports sign-in with Google OAuth (Better Auth callback: `/api/auth/better/callback/google`).
- Uses a personal package name, repository, and homepage in `package.json`.
- (Add any further changes you make here.)

All upstream features below are kept.

## 🚀 Key Features

- **Enhanced Xiaozhi Integration**:
  - **Multi-endpoint management**: Bidirectional WebSocket connections with multiple Xiaozhi endpoints (enable/disable, edit, reconnect, status query).
  - **Smart reconnection**: Fast reconnect mode, exponential backoff, infinite retries with sleep window, all configurable via environment variables.
  - **Tool synchronization**: Xiaozhi endpoints are notified when server tools change to keep tool lists up-to-date.
  - **Grouping and smart routing**: Endpoints can bind to groups or use `$smart` intelligent routing.

- **MCP Management (inherited and enhanced)**:
  - Manage standard MCP servers (stdio / SSE / HTTP modes).
  - Enable/disable servers, tools, and prompts with descriptions and grouping.
  - Unified MCP entry, group entry, and server-specific endpoints.

- **Console & Authentication**:
  - Frontend console (React + Vite + Tailwind) to manage Servers, Groups, Users, Logs, Settings, Xiaozhi Endpoints, and Market.
  - JWT-based authentication with user context middleware and a built-in admin account.

- **OpenAPI & Direct Tool Access**:
  - Expose OpenAPI documentation and statistics endpoints.
  - Call specific server tools via OpenAPI-compatible endpoints.

## 🔧 Quick Start

### Method 1: Docker Compose (recommended)

This repository ships with `docker-compose.yml` including both `pgvector` and the app. The app image is built locally from the `Dockerfile`.

1. Create a `.env` file next to `docker-compose.yml` (never commit it):

   ```env
   JWT_SECRET=replace-with-a-long-random-string
   DB_PASSWORD=replace-with-your-db-password
   # Google OAuth (only if you use Google sign-in)
   # Check the variable names expected by the code before filling these in.
   GOOGLE_CLIENT_ID=your-client-id
   GOOGLE_CLIENT_SECRET=your-client-secret
   ```

2. Build and start:

   ```bash
   docker compose up -d --build

   # View logs (optional)
   docker compose logs -f mcphub
   ```

   After the first build, `docker compose up -d` is enough. Use `--build` again only after changing source code or the `Dockerfile`.

3. Open the dashboard at <http://localhost:3000>.

Key variables (edit in compose or `.env` if needed):

- `DB_URL`: `postgres://xiaozhi:<password>@db:5432/xiaozhi_mcphub` (legacy `DATABASE_URL` still works)
- `JWT_SECRET`: JWT secret (required in this fork).
- `SMART_ROUTING_ENABLED`: Enable/disable smart routing (default "false").
- Optional: `BASE_PATH`, `OPENAI_API_KEY`, `OPENAI_API_BASE_URL`, `OPENAI_API_EMBEDDING_MODEL` (the OpenAI variables are required when enabling smart routing).

Default admin: `admin` / `admin123` (please change after first login).

### Google OAuth setup

1. In Google Cloud Console, open **Google Auth Platform**, create an **OAuth client** of type **Web application**.
2. Add this **Authorized redirect URI** (it must match exactly):

   ```
   http://localhost:3000/api/auth/better/callback/google
   ```

3. While the app is in **Testing** status, add your Google account under **Audience → Test users**.
4. Put the Client ID and Client Secret in `.env`, then restart with `docker compose up -d`.

### Method 2: Local development

Requirements: Node.js 24+, pnpm, PostgreSQL 16+ (recommended to use the `db` service from the repo's compose).

```bash
git clone https://github.com/hdkp81/xiaozhi-mcphub-hdkp.git
cd xiaozhi-mcphub-hdkp
pnpm install

# Start local database (optional, reuse compose's db)
docker compose up -d db

# Set database connection (or write to .env)
export DB_URL="postgres://xiaozhi:<password>@localhost:5432/xiaozhi_mcphub"

# Start both backend (:3000) and frontend (Vite :5173)
pnpm dev
```

Access the frontend dev server at `http://localhost:5173` (the frontend proxies to backend `:3000`).

### Upgrading from v1.0.3

If you already jumped to 1.1.x and servers disappeared, **do not wipe the Postgres volume**.
1.1.x+ will detect the legacy `mcp_servers` table (when still present) and copy any missing rows into the new `servers` table on startup. Prefer renaming the env var:

```bash
# before (v1.0.3)
DATABASE_URL=postgres://xiaozhi:…@db:5432/xiaozhi_mcphub

# after (1.1.x), same credentials / same volume
DB_URL=postgres://xiaozhi:…@db:5432/xiaozhi_mcphub
```

Details: [documents/docs/configuration/upgrade-from-v103.md](documents/docs/configuration/upgrade-from-v103.md).

## 🔑 Resetting the Admin Password

Passwords are stored hashed, so they cannot be read back. To reset the built-in admin:

```bash
docker compose exec db psql -U xiaozhi -d xiaozhi_mcphub -c "DELETE FROM users WHERE username = 'admin';"
docker compose restart mcphub
```

Then sign in with `admin` / `admin123` and change the password immediately.

## 🗺️ Smart Routing (optional)

Set `SMART_ROUTING_ENABLED` to `true` and provide `OPENAI_API_KEY` to enable it. The system uses `pgvector` for vector storage and indexing. If no vectors exist, index building will be skipped and later populated by vector services.

## 🖥️ Console Features (Frontend)

- Dashboard: Overview and status
- Servers: Server and tool management
- Groups: Grouping and assignment
- Users: User and permissions (admin)
- Logs: Real-time and historical logs
- Settings: System configuration (including smart routing)
- Xiaozhi Endpoints: Xiaozhi endpoint management
- Market: Search and install MCP servers from the community

## 📦 Defaults & Initialization

- Default admin: `admin` / `admin123`
- Default MCP servers: amap / playwright / fetch / slack (can be modified in the console and configured via ENV)

## 📄 License (Apache License 2.0)

This project is a modified version of [xiaozhi-mcphub](https://github.com/huangjunsen0406/xiaozhi-mcphub) and is licensed under the **Apache License 2.0**. Keep `LICENSE` and `NOTICE` when redistributing, and document any modifications (see "Changes in This Fork" above). See [`LICENSE`](LICENSE) and [`NOTICE`](NOTICE).

## 🤝 Contributing

Contributions are welcome! Please open issues/PRs for improvements.

## 🔗 Links

- This fork: <https://github.com/hdkp81/xiaozhi-mcphub-hdkp>
- Original project: <https://github.com/huangjunsen0406/xiaozhi-mcphub>
- Vietnamese localization: <https://github.com/hdkp81/xiaozhi-mcphub-vn>
- Xiaozhi AI Platform: <https://xiaozhi.me>
- Model Context Protocol: <https://modelcontextprotocol.io>