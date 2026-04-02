# n8n + PostgreSQL Docker Setup

Ephemeral n8n workflow automation platform with PostgreSQL database backend for development and testing in codespace.

## Quick Start

### 1. Generate Encryption Key (First Time Only)
```bash
openssl rand -hex 32
```

### 2. Configure Environment
```bash
# Copy the example and update with your values
cp .env.example .env.local

# Edit .env.local and paste the generated encryption key into N8N_ENCRYPTION_KEY
nano .env.local  # or your preferred editor
```

### 3. Start Services
```bash
docker-compose up -d
```

### 4. Verify Services
```bash
docker-compose ps
```

Both services should show status `Up`.

### 5. Access n8n UI
Open your browser and navigate to:
```
http://localhost:5678
```

## Service Details

### n8n
- **Image**: `docker.n8n.io/n8nio/n8n:latest`
- **Port**: 5678
- **UI**: http://localhost:5678
- **Health Check**: Automated via `/healthz` endpoint

### PostgreSQL
- **Image**: `postgres:16-alpine`
- **Port**: 5432
- **Database**: n8n
- **Default User**: n8n
- **Default Password**: n8n_secure_password_123 (change in .env.local)

## File Structure
```
.
├── docker-compose.yml      # Service orchestration
├── .env.local             # Secrets (generated, not version-controlled)
├── .env.example           # Configuration template
├── .gitignore             # Git ignore rules (includes .env.local)
└── README.md              # This file
```

## Common Commands

### Start Services
```bash
docker-compose up -d
```

### Stop Services
```bash
docker-compose down
```

### View Logs
```bash
# All services
docker-compose logs -f

# Specific service
docker-compose logs -f n8n
docker-compose logs -f postgres
```

### Check Service Health
```bash
# All services status
docker-compose ps

# PostgreSQL connectivity
docker-compose exec postgres psql -U n8n -d n8n -c "SELECT version();"

# n8n health
curl http://localhost:5678/healthz
```

### Database Access
```bash
# Connect to PostgreSQL directly
docker-compose exec postgres psql -U n8n -d n8n

# List tables
docker-compose exec postgres psql -U n8n -d n8n -c "\dt"
```

## Troubleshooting

### n8n fails to start
```bash
# Check logs
docker-compose logs n8n

# Common issue: PostgreSQL not ready yet
# Solution: Wait a few seconds and retry
```

### PostgreSQL connection issues
```bash
# Verify PostgreSQL is running
docker-compose logs postgres

# Test connection
docker-compose exec postgres pg_isready -U n8n
```

### Services not starting
```bash
# Ensure .env.local exists and N8N_ENCRYPTION_KEY is set
ls -la .env.local

# Rebuild images
docker-compose up -d --build
```

### Port conflicts
If port 5678 or 5432 is already in use:
1. Edit `docker-compose.yml`
2. Change the port mapping: `"5678:5678"` → `"YOUR_PORT:5678"`
3. Restart: `docker-compose restart`

## Environment Variables

All configuration is managed through `.env.local`. Key variables:

| Variable | Purpose | Default |
|----------|---------|---------|
| `N8N_ENCRYPTION_KEY` | Encrypts n8n credentials | (generated) |
| `POSTGRES_PASSWORD` | PostgreSQL password | n8n_secure_password_123 |
| `NODE_ENV` | Node environment | development |
| `N8N_PORT` | n8n listening port | 5678 |
| `N8N_DIAGNOSTICS_ENABLED` | Send diagnostics data | false |

## Ephemeral Design

⚠️ **Important**: This setup is ephemeral. All data is **lost** when containers are stopped/removed.

- No persistent volumes
- Fresh database on each startup
- Useful for: development, testing, CI/CD pipelines
- **Not suitable for production data**

## Data Persistence (Optional)

If you need data to persist between restarts, uncomment volumes in `docker-compose.yml`:

```yaml
postgres:
  volumes:
    - postgres_data:/var/lib/postgresql/data

volumes:
  postgres_data:
```

## Security Notes

- `.env.local` is **not version-controlled** (see .gitignore)
- Never commit secrets to Git
- Regenerate `N8N_ENCRYPTION_KEY` for new environments
- Change default PostgreSQL password in production
- Use strong passwords for real workflows

## Network

Services communicate via internal Docker bridge network `n8n_network`:
- n8n connects to PostgreSQL at `postgres:5432`
- External access through mapped ports only

## Requirements

- Docker
- Docker Compose
- ~500MB free disk space
- Ports 5678 and 5432 available

## Next Steps

1. Start the services: `docker-compose up -d`
2. Access n8n at http://localhost:5678
3. Create your first workflow
4. Test database persistence by creating a test workflow with PostgreSQL data
