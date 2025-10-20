# Stack Validation Patterns

This document outlines the architecture patterns and validation criteria for GitLab stack projects.

## Table of Contents
1. [Directory Structure Patterns](#directory-structure-patterns)
2. [Environment Variable Patterns](#environment-variable-patterns)
3. [Secrets Management Patterns](#secrets-management-patterns)
4. [Docker Compose Patterns](#docker-compose-patterns)
5. [Configuration Patterns](#configuration-patterns)
6. [File Ownership Patterns](#file-ownership-patterns)

---

## Directory Structure Patterns

### Standard Stack Directory Layout

```
my-stack/
├── docker-compose.yml           # Main compose file (NO version field)
├── .env                         # Environment variables (NOT in git)
├── .env.example                 # Environment template (IN git)
├── .gitignore                   # Must exclude secrets, .env, _temporary
├── .stack-validator.yml         # Optional: Custom validation rules
├── config/                      # Configuration files
│   ├── nginx/
│   │   └── nginx.conf
│   ├── app/
│   │   └── settings.yml
│   └── db/
│       └── init.sql
├── secrets/                     # Secret files (NOT in git)
│   ├── db_password
│   ├── api_key
│   └── jwt_secret
└── _temporary/                  # Transient files (NOT in git)
    └── (cleaned after use)
```

### Required Directories

| Directory | Purpose | Git Status | Permissions |
|-----------|---------|------------|-------------|
| `./config` | Configuration files | Tracked | 755 |
| `./secrets` | Secret files | **NOT tracked** | 700 |
| `./_temporary` | Temporary/cache files | **NOT tracked** | 755 |

### .gitignore Requirements

**MUST contain:**
```gitignore
# Secrets - never commit
/secrets/
/secrets/*

# Environment variables - never commit
.env

# Temporary files
/_temporary/
/_temporary/*

# Common exclusions
*.log
.DS_Store
```

---

## Environment Variable Patterns

### .env File Structure

**Purpose**: Define environment-specific variables for stack deployment

**Example .env:**
```bash
# Application
APP_NAME=my-application
APP_ENV=production
APP_DEBUG=false
APP_URL=https://example.com

# Database
DB_HOST=postgres
DB_PORT=5432
DB_NAME=app_database
DB_USER=app_user
# NOTE: DB_PASSWORD should be in ./secrets/db_password, not here!

# Redis
REDIS_HOST=redis
REDIS_PORT=6379

# Ports
WEB_PORT=80
API_PORT=8080

# Docker
COMPOSE_PROJECT_NAME=my-stack
```

### .env.example File Structure

**Purpose**: Template for required environment variables

**CRITICAL RULE**: .env.example MUST contain ALL variables from .env and vice versa

**Example .env.example:**
```bash
# Application
APP_NAME=my-application
APP_ENV=development
APP_DEBUG=true
APP_URL=http://localhost

# Database
DB_HOST=postgres
DB_PORT=5432
DB_NAME=app_database
DB_USER=app_user
# NOTE: DB_PASSWORD is managed via Docker secrets in ./secrets/

# Redis
REDIS_HOST=redis
REDIS_PORT=6379

# Ports - Customize for your environment
WEB_PORT=80
API_PORT=8080

# Docker
COMPOSE_PROJECT_NAME=my-stack
```

### Environment Variable Validation Rules

1. **Synchronization**: Every variable in .env MUST be in .env.example
2. **Documentation**: .env.example should have comments explaining each variable
3. **No Secrets**: .env should NOT contain passwords, API keys, tokens, or secrets
4. **Default Values**: .env.example should have safe defaults for development
5. **Required Variables**: Both files must define all required variables

### Variables That Should NOT Be in .env

Move these to ./secrets and use Docker secrets:

```bash
# ❌ BAD - Don't put these in .env
DB_PASSWORD=supersecret123
API_KEY=sk_live_abc123xyz
JWT_SECRET=my-jwt-secret-key
STRIPE_SECRET_KEY=sk_test_123
OAUTH_CLIENT_SECRET=abc123xyz

# ✅ GOOD - Reference via Docker secrets instead
# See docker-compose.yml secrets section
# Secrets are in ./secrets/ directory
```

---

## Secrets Management Patterns

### Secrets Directory Structure

```
secrets/
├── db_password              # PostgreSQL password
├── db_root_password         # Root password (if needed)
├── api_key                  # External API key
├── jwt_secret              # JWT signing secret
└── oauth_client_secret     # OAuth secret
```

### Secret File Format

**Single-line, no trailing newline:**
```bash
# Create secret (no newline)
echo -n "my-secret-value" > ./secrets/db_password

# ✅ Correct: 16 bytes
# ❌ Wrong: 17 bytes (includes newline)
```

### Secret File Permissions

```bash
# Directory
chmod 700 ./secrets/

# Individual files
chmod 600 ./secrets/db_password
chmod 600 ./secrets/api_key
```

### Docker Compose Secrets Pattern

**Top-level secrets definition:**
```yaml
secrets:
  db_password:
    file: ./secrets/db_password
  api_key:
    file: ./secrets/api_key
  jwt_secret:
    file: ./secrets/jwt_secret
```

**Service secrets reference:**
```yaml
services:
  app:
    image: myapp:latest
    secrets:
      - db_password
      - api_key
      - jwt_secret
    environment:
      # ✅ GOOD - Reference location, not value
      DB_PASSWORD_FILE: /run/secrets/db_password
      API_KEY_FILE: /run/secrets/api_key

      # ❌ BAD - Don't put actual secrets here
      # DB_PASSWORD: supersecret123
```

### Application Secret Usage

**In application code:**
```python
# Read secret from Docker secrets mount
def get_secret(secret_name):
    secret_path = f'/run/secrets/{secret_name}'
    with open(secret_path, 'r') as f:
        return f.read().strip()

# Usage
db_password = get_secret('db_password')
api_key = get_secret('api_key')
```

---

## Docker Compose Patterns

### Modern Docker Compose Format

**❌ DON'T use version field:**
```yaml
# ❌ OLD - Don't include version
version: '3.8'
```

**✅ DO use modern format:**
```yaml
# ✅ MODERN - No version field
services:
  app:
    image: myapp:latest
```

### Complete Stack Example

```yaml
services:
  app:
    image: myapp:latest
    container_name: my-app
    restart: unless-stopped
    depends_on:
      - postgres
      - redis
    secrets:
      - db_password
      - api_key
    environment:
      APP_ENV: ${APP_ENV}
      DB_HOST: ${DB_HOST}
      DB_PORT: ${DB_PORT}
      DB_NAME: ${DB_NAME}
      DB_USER: ${DB_USER}
      DB_PASSWORD_FILE: /run/secrets/db_password
      API_KEY_FILE: /run/secrets/api_key
    volumes:
      - ./config/app:/app/config:ro
      - app-data:/app/data
    networks:
      - app-network
    ports:
      - "${WEB_PORT}:80"

  postgres:
    image: postgres:16-alpine
    container_name: my-postgres
    restart: unless-stopped
    secrets:
      - db_password
    environment:
      POSTGRES_DB: ${DB_NAME}
      POSTGRES_USER: ${DB_USER}
      POSTGRES_PASSWORD_FILE: /run/secrets/db_password
    volumes:
      - ./config/db/init.sql:/docker-entrypoint-initdb.d/init.sql:ro
      - postgres-data:/var/lib/postgresql/data
    networks:
      - app-network

  redis:
    image: redis:7-alpine
    container_name: my-redis
    restart: unless-stopped
    volumes:
      - redis-data:/data
    networks:
      - app-network

volumes:
  app-data:
  postgres-data:
  redis-data:

networks:
  app-network:
    driver: bridge

secrets:
  db_password:
    file: ./secrets/db_password
  api_key:
    file: ./secrets/api_key
```

### Volume Mount Patterns

**Configuration files (read-only):**
```yaml
volumes:
  - ./config/nginx/nginx.conf:/etc/nginx/nginx.conf:ro
  - ./config/app/settings.yml:/app/config/settings.yml:ro
```

**Secrets (via Docker secrets - automatic mount):**
```yaml
secrets:
  - db_password  # Mounted at /run/secrets/db_password
```

**Persistent data (named volumes):**
```yaml
volumes:
  - postgres-data:/var/lib/postgresql/data
  - redis-data:/data
```

**Temporary files (local directory):**
```yaml
volumes:
  - ./_temporary/cache:/app/cache
  - ./_temporary/uploads:/app/uploads
```

---

## Configuration Patterns

### Configuration File Organization

**By service:**
```
config/
├── nginx/
│   ├── nginx.conf
│   └── ssl/
│       ├── cert.pem
│       └── key.pem
├── app/
│   ├── settings.yml
│   └── logging.conf
└── db/
    └── init.sql
```

### Configuration vs Secrets Separation

**✅ Configuration (in ./config):**
- Server hostnames
- Port numbers
- Feature flags
- Logging levels
- Public certificates
- Database names
- Cache settings

**❌ NOT Configuration (in ./secrets):**
- Passwords
- API keys
- Tokens
- Private keys
- OAuth secrets
- JWT secrets
- Encryption keys

### Example Configuration File

**config/app/settings.yml:**
```yaml
# Application Settings
app:
  name: ${APP_NAME}
  environment: ${APP_ENV}
  debug: ${APP_DEBUG}
  url: ${APP_URL}

# Database (connection info, NOT credentials)
database:
  host: ${DB_HOST}
  port: ${DB_PORT}
  name: ${DB_NAME}
  user: ${DB_USER}
  # Password loaded from /run/secrets/db_password

# Redis
cache:
  driver: redis
  host: ${REDIS_HOST}
  port: ${REDIS_PORT}

# Logging
logging:
  level: info
  output: stdout
  format: json
```

---

## File Ownership Patterns

### Correct Ownership

All stack files should be owned by the Docker user (current user), NOT root.

**Check ownership:**
```bash
# List all files with ownership
eza -la --tree

# Find root-owned files (should return nothing)
find . -type f -user root 2>/dev/null
```

### Common Ownership Issues

**Problem**: Files created by Docker containers as root

**Example scenario:**
```yaml
# Container runs as root, creates files in mounted volume
services:
  app:
    image: nginx:latest  # Runs as root by default
    volumes:
      - ./config/nginx.conf:/etc/nginx/nginx.conf
      - ./_temporary/cache:/var/cache/nginx  # ⚠️ Creates files as root!
```

**Fix**: Ensure containers run as non-root user

```yaml
services:
  app:
    image: nginx:latest
    user: "${UID}:${GID}"  # Run as current user
    volumes:
      - ./config/nginx.conf:/etc/nginx/nginx.conf
      - ./_temporary/cache:/var/cache/nginx
```

### Fixing Ownership

**Stack-validator detects ownership issues but doesn't fix them.**

**Manual fix (user action):**
```bash
# Fix ownership of specific file
sudo chown $(id -u):$(id -g) ./config/nginx.conf

# Fix ownership of entire directory
sudo chown -R $(id -u):$(id -g) ./config/

# Fix ownership of all project files
sudo chown -R $(id -u):$(id -g) .
```

**Prevention**: Use stack-creator skill to properly initialize stacks with correct ownership from the start.

---

## Validation Checklist

### Pre-Deployment Validation

Use this checklist to ensure stack readiness:

- [ ] **Directory Structure**
  - [ ] ./config directory exists
  - [ ] ./secrets directory exists with 700 permissions
  - [ ] ./_temporary directory exists
  - [ ] .gitignore excludes secrets, .env, _temporary

- [ ] **Environment Variables**
  - [ ] .env file exists and is valid
  - [ ] .env.example exists and is valid
  - [ ] .env and .env.example are synchronized
  - [ ] No secrets in .env file
  - [ ] .env is in .gitignore

- [ ] **Docker Configuration**
  - [ ] docker-compose.yml has no version field
  - [ ] docker-compose.yml passes docker-validation
  - [ ] Secrets defined in top-level secrets section
  - [ ] Services reference secrets via secrets key
  - [ ] Volume mounts follow patterns

- [ ] **Secrets Management**
  - [ ] All secret files exist in ./secrets
  - [ ] Secret files have 600 permissions
  - [ ] ./secrets directory has 700 permissions
  - [ ] No secrets in docker-compose.yml environment
  - [ ] No secrets in git

- [ ] **File Ownership**
  - [ ] No root-owned files in project
  - [ ] All files owned by Docker user
  - [ ] Config files have correct ownership

- [ ] **Configuration**
  - [ ] Config files properly organized
  - [ ] No secrets in config files
  - [ ] Config file syntax valid

---

## Reference: Common Validation Failures

| Issue | Category | Severity | Fix With |
|-------|----------|----------|----------|
| Missing .env.example | Environment | Critical | stack-creator |
| .env/.env.example mismatch | Environment | Critical | Manual sync + stack-creator |
| Secrets in .env | Security | Critical | secrets-manager |
| ./secrets not in .gitignore | Security | Critical | stack-creator |
| Root-owned files | Ownership | High | Manual chown |
| Missing ./secrets directory | Secrets | High | stack-creator |
| docker-compose.yml has version | Docker | Medium | Manual edit |
| Secrets in environment vars | Security | Critical | secrets-manager |
| Missing required directory | Structure | High | stack-creator |
| ./_temporary not empty | Cleanup | Low | Manual cleanup |

---

*These patterns ensure consistent, secure, and maintainable GitLab stack projects.*
