# PostgreSQL Container Setup & Connection Guide

Please refer to the full setup guide in [database-studies/rdbms/db-setup.md](../rdbms/db-setup.md).

For quick reference:

```bash
# Start PostgreSQL Container
docker compose -f .devcontainer/docker-compose.yml up -d db

# Connect via interactive psql shell
docker compose -f .devcontainer/docker-compose.yml exec -it db psql -U postgres -d postgres

# Check database readiness
docker compose -f .devcontainer/docker-compose.yml exec db pg_isready -U postgres
```

### Credentials:
- **Host:** `localhost`
- **Port:** `5432`
- **User:** `postgres`
- **Password:** `postgres`
- **Database:** `postgres`
