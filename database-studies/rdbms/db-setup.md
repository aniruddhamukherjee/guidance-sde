# PostgreSQL Container Setup & Connection Guide

This guide covers everything you need to start, manage, and connect to the PostgreSQL database container in this Codespace—both from inside the environment and from an external SQL client on your local machine.

---

## 1. Quick Connection Reference

| Parameter | Value |
|---|---|
| **Host** | `localhost` (or `127.0.0.1`) |
| **Port** | `5432` |
| **User** | `postgres` |
| **Password** | `postgres` |
| **Database** | `postgres` |
| **Connection URI** | `postgresql://postgres:postgres@localhost:5432/postgres` |

---

## 2. Configuration Overview

The database container is defined in the project's dev container setup:
- **[.devcontainer/docker-compose.yml](file:///workspaces/guidance-sde/.devcontainer/docker-compose.yml)**: Defines the `db` service using the official `postgres:latest` image, maps port `5432:5432`, and mounts a persistent volume `postgres-data`.
- **[.devcontainer/devcontainer.json](file:///workspaces/guidance-sde/.devcontainer/devcontainer.json)**: Configured with `"forwardPorts": [5432]` so VS Code automatically forwards the port.

---

## 3. Starting & Managing the Container

Run these commands from the root directory (`/workspaces/guidance-sde`):

### Start the PostgreSQL Container
```bash
docker compose -f .devcontainer/docker-compose.yml up -d db
```

### Verify Container Status
```bash
docker ps
```
You should see `devcontainer-db-1` with status `Up` and port mapping `0.0.0.0:5432->5432/tcp`.

### Check Database Readiness
```bash
docker compose -f .devcontainer/docker-compose.yml exec db pg_isready -U postgres
```
Expected output:
```text
/var/run/postgresql:5432 - accepting connections
```

### View Logs
```bash
docker compose -f .devcontainer/docker-compose.yml logs db --tail=50
```

### Stop or Restart
- **Stop container:** `docker compose -f .devcontainer/docker-compose.yml stop db`
- **Restart container:** `docker compose -f .devcontainer/docker-compose.yml restart db`
- **Stop and remove containers (data in volume is preserved):** `docker compose -f .devcontainer/docker-compose.yml down`

---

## 4. Connecting from Inside the Codespace

### Option A: Interactive `psql` Shell (No Client Installation Required)
Drop directly into the container's built-in `psql` CLI:

```bash
docker compose -f .devcontainer/docker-compose.yml exec -it db psql -U postgres -d postgres
```
*(Type `\q` and press Enter to exit)*

### Option B: Install `psql` in the Codespace Terminal
If you want to run `psql` directly without docker commands:

```bash
# 1. Install client package
sudo apt-get update && sudo apt-get install -y postgresql-client

# 2. Connect
export PGPASSWORD=postgres
psql -h localhost -U postgres -d postgres
```

### Option C: Execute SQL Queries or Files Directly
```bash
# Run a one-off query:
docker compose -f .devcontainer/docker-compose.yml exec db psql -U postgres -d postgres -c "SELECT version();"

# Run a .sql file:
docker compose -f .devcontainer/docker-compose.yml exec -T db psql -U postgres -d postgres < path/to/script.sql
```

### Option D: Connect via Python Code
```bash
pip install psycopg2-binary
```

```python
import psycopg2

conn = psycopg2.connect("postgresql://postgres:postgres@localhost:5432/postgres")
with conn.cursor() as cur:
    cur.execute("SELECT version();")
    print(cur.fetchone())
conn.close()
```

### Option E: VS Code Database Extension (GUI)
1. Install an extension like **Database Client** or **PostgreSQL** (by Chris Kolkman) from the Extensions tab (`Ctrl+Shift+X`).
2. Add a new PostgreSQL connection with Host: `localhost`, Port: `5432`, User: `postgres`, Password: `postgres`, Database: `postgres`.

---

## 5. Connecting from Your Local Machine SQL Client (DBeaver, TablePlus, pgAdmin, DataGrip)

Because PostgreSQL uses raw TCP sockets, connecting from your personal machine depends on how you opened this Codespace:

### If You Use VS Code Desktop (Recommended)
1. Open the Codespace in VS Code Desktop:
   - Click the **Application Menu** (top-left ☰) $\rightarrow$ **Open in VS Code Desktop** (or Command Palette: `Codespaces: Open in VS Code Desktop`).
2. VS Code Desktop automatically bridges forwarded ports to your physical machine's network.
3. Open your local SQL client (DBeaver, TablePlus, pgAdmin, etc.) and connect directly to:
   - **Host:** `localhost` (or `127.0.0.1`)
   - **Port:** `5432`
   - **User:** `postgres`
   - **Password:** `postgres`
   - **Database:** `postgres`

### If You Use Codespaces in the Web Browser
Browser-based Codespaces forward ports through HTTPS, which standard SQL desktop clients cannot communicate over directly. You can bridge raw TCP using the GitHub CLI:

1. In your **local machine's terminal** (not in the browser), run:
   ```bash
   gh codespace ports forward 5432:5432
   ```
2. Keep the command running.
3. Connect your local SQL client to `localhost:5432`.

---

## 6. Using GitHub Secrets for Production / CI Credentials

To avoid hardcoded passwords in `.devcontainer/docker-compose.yml`:

### Pass-through syntax (without fallback)
```yaml
  db:
    image: postgres:latest
    restart: unless-stopped
    ports:
      - "5432:5432"
    volumes:
      - postgres-data:/var/lib/postgresql
    environment:
      POSTGRES_USER: ${POSTGRES_USER}
      POSTGRES_DB: ${POSTGRES_DB}
      POSTGRES_PASSWORD: ${POSTGRES_PASSWORD}
```

### Mandatory syntax (fails immediately if secret is not set)
```yaml
    environment:
      POSTGRES_USER: ${POSTGRES_USER:?POSTGRES_USER is required}
      POSTGRES_DB: ${POSTGRES_DB:?POSTGRES_DB is required}
      POSTGRES_PASSWORD: ${POSTGRES_PASSWORD:?POSTGRES_PASSWORD is required}
```
