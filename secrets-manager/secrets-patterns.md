# Secrets Management Patterns

This document outlines the secure patterns for managing secrets in GitLab stack projects.

## Table of Contents
1. [Core Security Principles](#core-security-principles)
2. [Directory and File Structure](#directory-and-file-structure)
3. [Docker Secrets Integration](#docker-secrets-integration)
4. [Secret Detection Patterns](#secret-detection-patterns)
5. [Migration Patterns](#migration-patterns)
6. [Common Secret Types](#common-secret-types)

---

## Core Security Principles

### The Golden Rules

**NEVER**:
- ❌ Put secrets in .env files
- ❌ Put secrets in docker-compose.yml environment variables
- ❌ Commit secrets to git
- ❌ Use world-readable permissions
- ❌ Store secrets as root-owned files
- ❌ Hardcode secrets in application code
- ❌ Log secret values
- ❌ Pass secrets via command-line arguments

**ALWAYS**:
- ✅ Use Docker secrets mechanism
- ✅ Store secret files in ./secrets directory
- ✅ Set 700 permissions on ./secrets directory
- ✅ Set 600 permissions on secret files
- ✅ Add ./secrets/* to .gitignore
- ✅ Use cryptographically secure random generation
- ✅ Rotate secrets regularly
- ✅ Audit secret usage

---

## Directory and File Structure

### Standard ./secrets Directory Layout

```
./secrets/
├── .gitkeep                    # Only file tracked by git
├── db_password                 # Database password
├── db_root_password            # Database root password
├── api_key                     # External API key
├── jwt_secret                  # JWT signing secret
├── oauth_client_secret         # OAuth secret
├── smtp_password               # Email password
├── encryption_key              # Application encryption key
└── ssl/
    ├── cert.pem                # SSL certificate
    └── key.pem                 # SSL private key
```

### Permissions Reference

```bash
# Directory permissions
drwx------  ./secrets/                    # 700 (owner only)

# File permissions
-rw-------  db_password                   # 600 (owner read/write)
-rw-------  api_key                       # 600
-rw-------  jwt_secret                    # 600

# Ownership
user:user   all files and directories     # NOT root
```

### Setting Up Proper Permissions

```bash
# Create secrets directory
mkdir -p ./secrets
chmod 700 ./secrets

# Create secret file
echo -n "secret-value" > ./secrets/db_password
chmod 600 ./secrets/db_password

# Verify permissions
ls -la ./secrets/
# Expected: drwx------  ... ./secrets/
# Expected: -rw-------  ... db_password

# Fix ownership if root-owned
sudo chown -R $(id -u):$(id -g) ./secrets/
```

---

## Docker Secrets Integration

### Top-Level Secrets Definition

**File-based secrets** (preferred for development/single-host):

```yaml
secrets:
  db_password:
    file: ./secrets/db_password

  api_key:
    file: ./secrets/api_key

  jwt_secret:
    file: ./secrets/jwt_secret

  # SSL certificates
  ssl_cert:
    file: ./secrets/ssl/cert.pem

  ssl_key:
    file: ./secrets/ssl/key.pem
```

**External secrets** (for production/swarm):

```yaml
secrets:
  db_password:
    external: true
    name: prod_db_password

  api_key:
    external: true
    name: prod_api_key_v2
```

### Service Secret Usage

**Basic usage**:

```yaml
services:
  app:
    image: myapp:latest
    secrets:
      - db_password
      - api_key
      - jwt_secret
    # Secrets mounted at /run/secrets/secret_name
```

**Containers with native Docker secrets support**:

```yaml
services:
  postgres:
    image: postgres:16-alpine
    secrets:
      - db_password
    environment:
      POSTGRES_DB: myapp
      POSTGRES_USER: appuser
      # Use _FILE suffix to point to secret
      POSTGRES_PASSWORD_FILE: /run/secrets/db_password
```

**Supported containers with _FILE suffix**:
- PostgreSQL: `POSTGRES_PASSWORD_FILE`
- MySQL/MariaDB: `MYSQL_ROOT_PASSWORD_FILE`, `MYSQL_PASSWORD_FILE`
- MongoDB: Various `_FILE` variables
- Redis: Configuration file can read from secret path

**Containers requiring docker-entrypoint.sh**:

```yaml
services:
  custom_app:
    image: myapp:latest
    entrypoint: /docker-entrypoint.sh
    command: ["npm", "start"]
    volumes:
      - ./docker-entrypoint.sh:/docker-entrypoint.sh:ro
    secrets:
      - api_key
      - jwt_secret
    # docker-entrypoint.sh loads secrets into environment
```

---

## Secret Detection Patterns

### Patterns That Indicate Secrets

**Variable Name Patterns** (case-insensitive):

```regex
.*PASSWORD.*
.*SECRET.*
.*KEY.*
.*TOKEN.*
.*API.*
.*AUTH.*
.*CREDENTIAL.*
.*PRIVATE.*
.*CERT.*
```

**Value Patterns**:

```regex
# Base64-encoded (long strings)
^[A-Za-z0-9+/]{40,}={0,2}$

# Hex strings (64+ chars)
^[a-f0-9]{64,}$

# JWT tokens
^eyJ[A-Za-z0-9-_]+\.[A-Za-z0-9-_]+\.[A-Za-z0-9-_]+$

# API keys (common formats)
^sk_live_[A-Za-z0-9]{24,}$
^pk_live_[A-Za-z0-9]{24,}$

# UUID format
^[0-9a-f]{8}-[0-9a-f]{4}-[0-9a-f]{4}-[0-9a-f]{4}-[0-9a-f]{12}$
```

### Scanning .env for Secrets

**Examples of secrets in .env** (BAD):

```bash
# ❌ BAD - These are secrets and should NOT be in .env
DB_PASSWORD=supersecret123
API_KEY=sk_live_abc123xyz789
JWT_SECRET=my-super-secret-jwt-key
STRIPE_SECRET_KEY=sk_test_abc123
OAUTH_CLIENT_SECRET=oauth-secret-123
ENCRYPTION_KEY=aes256-key-here
ADMIN_PASSWORD=admin123
SMTP_PASSWORD=email-password
PRIVATE_KEY=-----BEGIN PRIVATE KEY-----
```

**What SHOULD be in .env** (GOOD):

```bash
# ✅ GOOD - Non-secret configuration
APP_NAME=myapp
APP_ENV=production
APP_DEBUG=false
APP_URL=https://example.com

# Database connection (NOT credentials)
DB_HOST=postgres
DB_PORT=5432
DB_NAME=myapp_production
DB_USER=appuser
# DB_PASSWORD is in ./secrets/db_password

# Redis
REDIS_HOST=redis
REDIS_PORT=6379

# Ports
WEB_PORT=80
API_PORT=8080

# Feature flags
ENABLE_CACHING=true
ENABLE_LOGGING=true
```

### Scanning docker-compose.yml for Secrets

**Bad patterns** (secrets in environment):

```yaml
# ❌ BAD - Secrets in environment variables
services:
  app:
    environment:
      DB_PASSWORD: supersecret123              # ❌ CRITICAL
      API_KEY: sk_live_abc123                  # ❌ CRITICAL
      JWT_SECRET: my-jwt-secret                # ❌ CRITICAL

  postgres:
    environment:
      POSTGRES_PASSWORD: dbpassword123         # ❌ CRITICAL
```

**Good patterns** (using Docker secrets):

```yaml
# ✅ GOOD - Using Docker secrets
services:
  app:
    secrets:
      - db_password
      - api_key
      - jwt_secret
    environment:
      # Only non-secret config in environment
      DB_HOST: postgres
      DB_PORT: 5432
      DB_NAME: myapp

  postgres:
    secrets:
      - db_password
    environment:
      POSTGRES_DB: myapp
      POSTGRES_USER: appuser
      POSTGRES_PASSWORD_FILE: /run/secrets/db_password

secrets:
  db_password:
    file: ./secrets/db_password
  api_key:
    file: ./secrets/api_key
  jwt_secret:
    file: ./secrets/jwt_secret
```

---

## Migration Patterns

### Pattern 1: Migrate from .env to Docker Secrets

**Before** (.env file):

```bash
DB_PASSWORD=mysecretpass
API_KEY=sk_live_abc123xyz
JWT_SECRET=my-jwt-secret-key
```

**Migration steps**:

```bash
# 1. Create secret files
echo -n "mysecretpass" > ./secrets/db_password
echo -n "sk_live_abc123xyz" > ./secrets/api_key
echo -n "my-jwt-secret-key" > ./secrets/jwt_secret

# 2. Set permissions
chmod 600 ./secrets/*

# 3. Remove from .env
sed -i '/DB_PASSWORD=/d' .env
sed -i '/API_KEY=/d' .env
sed -i '/JWT_SECRET=/d' .env
```

**After** (.env file):

```bash
# Secrets moved to Docker secrets in ./secrets/
# DB_PASSWORD: ./secrets/db_password
# API_KEY: ./secrets/api_key
# JWT_SECRET: ./secrets/jwt_secret
```

### Pattern 2: Migrate from docker-compose.yml environment to Secrets

**Before**:

```yaml
services:
  app:
    environment:
      DB_HOST: postgres
      DB_PASSWORD: supersecret123    # ❌ Secret in compose
      API_KEY: sk_live_abc123        # ❌ Secret in compose
```

**Migration**:

```bash
# Extract values and create secret files
echo -n "supersecret123" > ./secrets/db_password
echo -n "sk_live_abc123" > ./secrets/api_key
chmod 600 ./secrets/*
```

**After**:

```yaml
services:
  app:
    secrets:
      - db_password
      - api_key
    environment:
      DB_HOST: postgres
      # Secrets loaded from /run/secrets/

secrets:
  db_password:
    file: ./secrets/db_password
  api_key:
    file: ./secrets/api_key
```

### Pattern 3: Create docker-entrypoint.sh for Legacy Containers

When container doesn't support `_FILE` variables:

**docker-entrypoint.sh**:

```bash
#!/bin/bash
set -e

# Function to load secret into environment variable
load_secret() {
  local secret_name=$1
  local env_var=$2
  local secret_file="/run/secrets/${secret_name}"

  if [ -f "$secret_file" ]; then
    export "${env_var}=$(cat "$secret_file")"
    echo "✓ Loaded secret: $secret_name -> $env_var"
  else
    echo "✗ ERROR: Secret file not found: $secret_file" >&2
    exit 1
  fi
}

# Load all required secrets
load_secret "db_password" "DB_PASSWORD"
load_secret "api_key" "API_KEY"
load_secret "jwt_secret" "JWT_SECRET"

# Execute the original command
exec "$@"
```

**docker-compose.yml**:

```yaml
services:
  legacy_app:
    image: legacy-app:latest
    entrypoint: /docker-entrypoint.sh
    command: ["node", "server.js"]
    volumes:
      - ./docker-entrypoint.sh:/docker-entrypoint.sh:ro
    secrets:
      - db_password
      - api_key
      - jwt_secret
```

**Set permissions**:

```bash
chmod +x docker-entrypoint.sh
```

---

## Common Secret Types

### 1. Database Passwords

**Generate**:
```bash
openssl rand -base64 32 | tr -d '/+=' | head -c 32 > ./secrets/db_password
chmod 600 ./secrets/db_password
```

**Use with PostgreSQL**:
```yaml
services:
  postgres:
    image: postgres:16-alpine
    secrets:
      - db_password
    environment:
      POSTGRES_PASSWORD_FILE: /run/secrets/db_password
```

### 2. API Keys

**Generate**:
```bash
openssl rand -hex 32 > ./secrets/api_key
chmod 600 ./secrets/api_key
```

**Format**: 64 hex characters

### 3. JWT Secrets

**Generate**:
```bash
openssl rand -base64 64 > ./secrets/jwt_secret
chmod 600 ./secrets/jwt_secret
```

**Format**: Base64-encoded, 64+ characters

### 4. Encryption Keys

**Generate AES-256 key**:
```bash
openssl rand -hex 32 > ./secrets/encryption_key
chmod 600 ./secrets/encryption_key
```

**Format**: 32 bytes hex (256-bit)

### 5. Session Secrets

**Generate**:
```bash
openssl rand -base64 32 > ./secrets/session_secret
chmod 600 ./secrets/session_secret
```

### 6. OAuth Client Secrets

**Usually provided by OAuth provider**, store securely:
```bash
echo -n "provider-given-secret" > ./secrets/oauth_client_secret
chmod 600 ./secrets/oauth_client_secret
```

### 7. SSL/TLS Certificates and Keys

**Store certificate and key separately**:

```bash
# Certificate (can be less restrictive)
cp cert.pem ./secrets/ssl/cert.pem
chmod 644 ./secrets/ssl/cert.pem

# Private key (must be restrictive)
cp key.pem ./secrets/ssl/key.pem
chmod 600 ./secrets/ssl/key.pem
```

**Use in compose**:
```yaml
secrets:
  ssl_cert:
    file: ./secrets/ssl/cert.pem
  ssl_key:
    file: ./secrets/ssl/key.pem

services:
  nginx:
    secrets:
      - ssl_cert
      - ssl_key
    # Mounted at /run/secrets/ssl_cert and /run/secrets/ssl_key
```

---

## Git Protection Patterns

### .gitignore Configuration

**Comprehensive .gitignore**:

```gitignore
# Secrets directory - NEVER commit
/secrets/
/secrets/*

# Allow only .gitkeep
!secrets/.gitkeep

# Backup files
*.old
*.backup
*.bak
*~

# Environment files (may contain secrets)
.env
.env.local
.env.*.local
.env.production

# Common secret file patterns
*password*.txt
*secret*.txt
*key*.txt
*token*.txt
*credential*.txt

# SSL/TLS
*.pem
*.key
*.crt
*.p12
*.pfx

# SSH keys
id_rsa
id_ed25519
*.ppk
```

### Checking Git Status

**Verify secrets aren't staged**:
```bash
# Check for secrets in staging
git status | grep secrets/

# Should only show .gitkeep if anything
# If other files shown, they're staged (BAD!)
```

**Check git history**:
```bash
# Search for secrets in history
git log --all --full-history -- ./secrets/

# Search for specific patterns
git log -p --all -S "password"
git log -p --all -S "secret"
```

**Remove secrets from git history** (if committed):
```bash
# Using git-filter-repo (recommended)
git filter-repo --path secrets/ --invert-paths

# Or BFG Repo-Cleaner
bfg --delete-folders secrets
```

---

## Secret Rotation Patterns

### Safe Rotation Procedure

```bash
# 1. Backup current secret
cp ./secrets/api_key ./secrets/api_key.$(date +%Y%m%d).old

# 2. Generate new secret
openssl rand -hex 32 > ./secrets/api_key

# 3. Test with new secret
docker compose up -d
docker compose logs app  # Check for errors

# 4. If successful, remove old backup after grace period
# rm ./secrets/api_key.*.old
```

### Rotation Tracking

**Create .secrets/metadata.yml** (not tracked):

```yaml
db_password:
  created: 2025-01-15
  rotated: 2025-10-20
  rotation_interval_days: 90

api_key:
  created: 2025-01-15
  rotated: 2025-10-20
  rotation_interval_days: 90
```

---

## Security Checklist

### Pre-Deployment

- [ ] All secrets in ./secrets directory
- [ ] Directory permissions: 700
- [ ] File permissions: 600
- [ ] No root-owned files
- [ ] ./secrets/* in .gitignore
- [ ] No secrets in .env
- [ ] No secrets in docker-compose.yml environment
- [ ] All referenced secrets exist
- [ ] docker-entrypoint.sh only when necessary
- [ ] No secrets in git history

### Post-Deployment

- [ ] Services can access secrets
- [ ] No secrets in container logs
- [ ] Secrets mounted at /run/secrets/
- [ ] No permission errors
- [ ] Rotation schedule established

---

*These patterns ensure secrets are managed securely throughout the stack lifecycle.*
