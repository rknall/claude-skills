# Secrets Migration Guide

This guide provides step-by-step instructions for migrating secrets from insecure locations (.env, docker-compose.yml environment variables) to secure Docker secrets.

## Table of Contents
1. [Why Migrate?](#why-migrate)
2. [Pre-Migration Checklist](#pre-migration-checklist)
3. [Migration Scenario 1: .env to Docker Secrets](#migration-scenario-1-env-to-docker-secrets)
4. [Migration Scenario 2: docker-compose.yml Environment to Docker Secrets](#migration-scenario-2-docker-composeyml-environment-to-docker-secrets)
5. [Migration Scenario 3: Combined Migration](#migration-scenario-3-combined-migration)
6. [Post-Migration Validation](#post-migration-validation)
7. [Troubleshooting](#troubleshooting)

---

## Why Migrate?

### Security Risks of Secrets in .env or docker-compose.yml

**Critical security issues**:
- 🔴 **Git exposure**: Files may be committed to version control
- 🔴 **World-readable**: Default permissions allow anyone on system to read
- 🔴 **Plaintext storage**: No encryption or protection
- 🔴 **Audit trail**: No tracking of who accessed secrets
- 🔴 **Rotation difficulty**: Hard to rotate without downtime
- 🔴 **Backup exposure**: Secrets copied in backups

### Benefits of Docker Secrets

- ✅ **Encrypted at rest and in transit** (in Swarm mode)
- ✅ **Never written to disk** in container filesystem
- ✅ **Mount-only access** at /run/secrets/
- ✅ **Proper permissions** automatically
- ✅ **Easy rotation** without code changes
- ✅ **Audit capabilities** built-in
- ✅ **Never in git** by design

---

## Pre-Migration Checklist

Before starting migration:

### 1. Backup Current Configuration

```bash
# Backup .env
cp .env .env.backup.$(date +%Y%m%d)

# Backup docker-compose.yml
cp docker-compose.yml docker-compose.yml.backup.$(date +%Y%m%d)

# Create migration log
echo "Migration started: $(date)" > .migration.log
```

### 2. Identify All Secrets

```bash
# Scan .env for potential secrets
grep -iE "(PASSWORD|SECRET|KEY|TOKEN|API|AUTH)" .env

# Scan docker-compose.yml for secrets in environment
grep -A 10 "environment:" docker-compose.yml | grep -iE "(PASSWORD|SECRET|KEY|TOKEN)"
```

### 3. Document Secret Usage

Create a migration plan:
```
Secret Name          | Current Location           | Service(s) Using
---------------------|----------------------------|------------------
DB_PASSWORD          | .env                       | postgres, app
API_KEY              | docker-compose.yml (app)   | app
JWT_SECRET           | .env                       | app
SMTP_PASSWORD        | docker-compose.yml (mail)  | mail
```

### 4. Ensure ./secrets Directory Exists

```bash
# Create if missing
mkdir -p ./secrets
chmod 700 ./secrets

# Add .gitkeep (only file that should be in git)
touch ./secrets/.gitkeep
git add ./secrets/.gitkeep
```

### 5. Verify .gitignore

```bash
# Add to .gitignore if not present
cat >> .gitignore << 'EOF'

# Secrets
/secrets/
/secrets/*
!secrets/.gitkeep
EOF
```

---

## Migration Scenario 1: .env to Docker Secrets

### Example: Database Password in .env

**Before** (.env):
```bash
DB_HOST=postgres
DB_PORT=5432
DB_NAME=myapp
DB_USER=appuser
DB_PASSWORD=supersecretpassword123  # ❌ Security risk!
```

### Step-by-Step Migration

**Step 1: Extract Secret Value**

```bash
# Read current value from .env
DB_PASSWORD=$(grep "^DB_PASSWORD=" .env | cut -d'=' -f2-)
echo "Found DB_PASSWORD in .env"
```

**Step 2: Create Secret File**

```bash
# Create secret file (no trailing newline!)
echo -n "$DB_PASSWORD" > ./secrets/db_password

# Set proper permissions
chmod 600 ./secrets/db_password

# Verify
ls -l ./secrets/db_password
# Expected: -rw------- ... db_password
```

**Step 3: Update docker-compose.yml**

Add secret definition:
```yaml
# Add to top-level secrets section
secrets:
  db_password:
    file: ./secrets/db_password
```

Update service to use secret:
```yaml
services:
  postgres:
    image: postgres:16-alpine
    secrets:
      - db_password
    environment:
      POSTGRES_DB: ${DB_NAME}
      POSTGRES_USER: ${DB_USER}
      # Use _FILE suffix for Docker secrets
      POSTGRES_PASSWORD_FILE: /run/secrets/db_password
```

**Step 4: Remove from .env**

```bash
# Comment out old value with migration note
sed -i 's/^DB_PASSWORD=.*/# DB_PASSWORD migrated to Docker secrets (\.\/secrets\/db_password)/' .env

# Or remove entirely
sed -i '/^DB_PASSWORD=/d' .env
```

**After** (.env):
```bash
DB_HOST=postgres
DB_PORT=5432
DB_NAME=myapp
DB_USER=appuser
# DB_PASSWORD migrated to Docker secrets (./secrets/db_password)
```

**Step 5: Update .env.example**

```bash
# Update .env.example to document the secret
cat >> .env.example << 'EOF'

# DB_PASSWORD is managed via Docker secrets
# File: ./secrets/db_password
# See README.md for secret setup instructions
EOF
```

**Step 6: Test**

```bash
# Restart services
docker compose down
docker compose up -d postgres

# Check logs for errors
docker compose logs postgres

# Verify secret is accessible inside container
docker compose exec postgres sh -c 'cat /run/secrets/db_password'
```

---

## Migration Scenario 2: docker-compose.yml Environment to Docker Secrets

### Example: API Key in docker-compose.yml

**Before**:
```yaml
services:
  app:
    image: myapp:latest
    environment:
      API_URL: https://api.example.com
      API_KEY: sk_live_abc123xyz789def456  # ❌ Security risk!
      APP_ENV: production
```

### Step-by-Step Migration

**Step 1: Extract Secret Value**

```bash
# Manually copy the value from docker-compose.yml
# API_KEY value: sk_live_abc123xyz789def456
```

**Step 2: Create Secret File**

```bash
# Create secret file
echo -n "sk_live_abc123xyz789def456" > ./secrets/api_key
chmod 600 ./secrets/api_key
```

**Step 3: Add Secret to docker-compose.yml**

```yaml
# Top-level secrets
secrets:
  api_key:
    file: ./secrets/api_key

services:
  app:
    image: myapp:latest
    secrets:
      - api_key
    environment:
      API_URL: https://api.example.com
      APP_ENV: production
      # If app supports reading from file
      API_KEY_FILE: /run/secrets/api_key
```

**Step 4: Application Code Update** (if needed)

If application doesn't support `_FILE` suffix:

**Option A**: Modify application to read from file

```javascript
// Node.js example
const fs = require('fs');

function getSecret(secretName) {
  try {
    return fs.readFileSync(`/run/secrets/${secretName}`, 'utf8').trim();
  } catch (err) {
    console.error(`Failed to read secret ${secretName}:`, err);
    process.exit(1);
  }
}

const apiKey = getSecret('api_key');
```

**Option B**: Use docker-entrypoint.sh

Create `docker-entrypoint.sh`:
```bash
#!/bin/bash
set -e

# Load API key from Docker secret
if [ -f /run/secrets/api_key ]; then
  export API_KEY=$(cat /run/secrets/api_key)
else
  echo "ERROR: api_key secret not found"
  exit 1
fi

# Execute original command
exec "$@"
```

Update docker-compose.yml:
```yaml
services:
  app:
    image: myapp:latest
    entrypoint: /docker-entrypoint.sh
    command: ["npm", "start"]
    volumes:
      - ./docker-entrypoint.sh:/docker-entrypoint.sh:ro
    secrets:
      - api_key
    environment:
      API_URL: https://api.example.com
      APP_ENV: production
```

```bash
chmod +x docker-entrypoint.sh
```

**Step 5: Remove from docker-compose.yml environment**

Remove the old API_KEY line:
```yaml
services:
  app:
    environment:
      API_URL: https://api.example.com
      APP_ENV: production
      # API_KEY removed - now using Docker secrets
```

**Step 6: Test**

```bash
docker compose up -d app
docker compose logs app

# Verify secret accessible
docker compose exec app sh -c 'cat /run/secrets/api_key'
```

---

## Migration Scenario 3: Combined Migration

### Example: Multiple Secrets in Both .env and docker-compose.yml

**Before**:

.env:
```bash
DB_PASSWORD=dbpass123
JWT_SECRET=jwt-secret-key-here
SMTP_PASSWORD=smtp-pass-123
```

docker-compose.yml:
```yaml
services:
  app:
    environment:
      DB_PASSWORD: ${DB_PASSWORD}
      JWT_SECRET: ${JWT_SECRET}
      API_KEY: sk_live_hardcoded123

  mail:
    environment:
      SMTP_PASSWORD: ${SMTP_PASSWORD}
```

### Comprehensive Migration

**Step 1: Create All Secret Files**

```bash
# Extract from .env
DB_PASSWORD=$(grep "^DB_PASSWORD=" .env | cut -d'=' -f2-)
JWT_SECRET=$(grep "^JWT_SECRET=" .env | cut -d'=' -f2-)
SMTP_PASSWORD=$(grep "^SMTP_PASSWORD=" .env | cut -d'=' -f2-)

# Create secret files
echo -n "$DB_PASSWORD" > ./secrets/db_password
echo -n "$JWT_SECRET" > ./secrets/jwt_secret
echo -n "$SMTP_PASSWORD" > ./secrets/smtp_password
echo -n "sk_live_hardcoded123" > ./secrets/api_key

# Set permissions
chmod 600 ./secrets/*

# Verify
ls -la ./secrets/
```

**Step 2: Update docker-compose.yml Completely**

```yaml
secrets:
  db_password:
    file: ./secrets/db_password
  jwt_secret:
    file: ./secrets/jwt_secret
  api_key:
    file: ./secrets/api_key
  smtp_password:
    file: ./secrets/smtp_password

services:
  app:
    image: myapp:latest
    secrets:
      - db_password
      - jwt_secret
      - api_key
    environment:
      # Only non-secret configuration
      DB_HOST: postgres
      APP_ENV: production

  mail:
    image: mailserver:latest
    secrets:
      - smtp_password
    environment:
      SMTP_HOST: smtp.example.com
      SMTP_PORT: 587
```

**Step 3: Create docker-entrypoint.sh** (if needed)

```bash
#!/bin/bash
set -e

# Load all secrets into environment
load_secret() {
  local secret_name=$1
  local env_var=$2

  if [ -f "/run/secrets/${secret_name}" ]; then
    export "${env_var}=$(cat "/run/secrets/${secret_name}")"
    echo "✓ Loaded ${env_var} from ${secret_name}"
  else
    echo "✗ Secret ${secret_name} not found!" >&2
    exit 1
  fi
}

# Load all required secrets
load_secret "db_password" "DB_PASSWORD"
load_secret "jwt_secret" "JWT_SECRET"
load_secret "api_key" "API_KEY"

exec "$@"
```

**Step 4: Clean Up .env**

```bash
# Remove all secrets from .env
sed -i '/^DB_PASSWORD=/d' .env
sed -i '/^JWT_SECRET=/d' .env
sed -i '/^SMTP_PASSWORD=/d' .env

# Add migration notes
cat >> .env << 'EOF'

# === SECRETS MIGRATED TO DOCKER SECRETS ===
# All sensitive credentials now in ./secrets/ directory
# - DB_PASSWORD: ./secrets/db_password
# - JWT_SECRET: ./secrets/jwt_secret
# - SMTP_PASSWORD: ./secrets/smtp_password
# - API_KEY: ./secrets/api_key
EOF
```

**Step 5: Update .env.example**

```bash
# .env.example should NOT have secret values
cat > .env.example << 'EOF'
# Application configuration
APP_ENV=development
DB_HOST=postgres
DB_PORT=5432

# === REQUIRED SECRETS ===
# Create these files in ./secrets/ directory:
# - db_password: Database password
# - jwt_secret: JWT signing secret
# - smtp_password: Email server password
# - api_key: External API key
#
# See README.md for instructions
EOF
```

**Step 6: Comprehensive Testing**

```bash
# Stop all services
docker compose down

# Start with new configuration
docker compose up -d

# Check all service logs
docker compose logs

# Verify secrets accessible in each container
docker compose exec app sh -c 'ls -la /run/secrets/'
docker compose exec mail sh -c 'ls -la /run/secrets/'

# Test application functionality
curl http://localhost:8080/health
```

---

## Post-Migration Validation

### Validation Checklist

Run these checks after migration:

```bash
# 1. No secrets in .env
! grep -iE "(password|secret|key|token)=[^ ]" .env

# 2. No secrets in docker-compose.yml environment
! grep -A 20 "environment:" docker-compose.yml | grep -iE "(password|secret|key|token).*:"

# 3. All secret files exist
for secret in db_password jwt_secret api_key smtp_password; do
  [ -f "./secrets/$secret" ] && echo "✓ $secret exists" || echo "✗ $secret MISSING"
done

# 4. Proper permissions
[ "$(stat -c '%a' ./secrets)" = "700" ] && echo "✓ Directory: 700" || echo "✗ Wrong permissions"
find ./secrets -type f ! -name .gitkeep -exec stat -c '%a %n' {} \; | while read perm file; do
  [ "$perm" = "600" ] && echo "✓ $file: 600" || echo "✗ $file: $perm (should be 600)"
done

# 5. Secrets not in git
! git ls-files | grep "secrets/" | grep -v .gitkeep

# 6. All services running
docker compose ps | grep Up
```

### Use stack-validator

```bash
# Run comprehensive validation
claude "validate this stack"

# Should show:
# ✅ No secrets in .env
# ✅ No secrets in docker-compose.yml environment
# ✅ All secret files exist with proper permissions
# ✅ ./secrets in .gitignore
```

---

## Troubleshooting

### Issue 1: Service Can't Access Secret

**Symptom**: Container logs show "permission denied" or "file not found"

**Diagnosis**:
```bash
# Check if secret is mounted
docker compose exec app ls -la /run/secrets/

# Check file permissions
ls -l ./secrets/db_password
```

**Fix**:
```bash
# Ensure proper permissions
chmod 600 ./secrets/db_password

# Ensure secret is defined in compose
grep -A 2 "secrets:" docker-compose.yml

# Restart service
docker compose restart app
```

### Issue 2: Application Still Expects Environment Variable

**Symptom**: Application error "Missing required environment variable"

**Solution A**: Use docker-entrypoint.sh
```bash
# Create entrypoint to load secrets into environment
# See Pattern 3 in secrets-patterns.md
```

**Solution B**: Modify application code
```javascript
// Read from /run/secrets/ instead of environment
const password = fs.readFileSync('/run/secrets/db_password', 'utf8').trim();
```

### Issue 3: Secrets Accidentally Committed to Git

**Symptom**: git status shows secrets/ files

**Immediate action**:
```bash
# DO NOT COMMIT!
git reset ./secrets/

# Ensure .gitignore is correct
echo "/secrets/*" >> .gitignore
echo "!secrets/.gitkeep" >> .gitignore

# Stage .gitignore
git add .gitignore
```

**If already committed**:
```bash
# Remove from git (keeps local file)
git rm --cached ./secrets/*
git add ./secrets/.gitkeep

# Commit the fix
git commit -m "Remove secrets from git (security fix)"

# IMPORTANT: Rotate all exposed secrets immediately!
# Anyone with access to git history can see them
```

**Clean git history** (if secrets were pushed):
```bash
# Use git-filter-repo (recommended)
git filter-repo --path secrets/ --invert-paths --force

# Force push (coordinate with team!)
git push --force
```

### Issue 4: Wrong Permissions After Docker Created Files

**Symptom**: Secret files owned by root

**Fix**:
```bash
# Change ownership
sudo chown $(id -u):$(id -g) ./secrets/*

# Fix permissions
chmod 700 ./secrets
chmod 600 ./secrets/*
```

### Issue 5: Secrets Work Locally but Not in Production

**Issue**: File-based secrets only work in docker compose, not Swarm

**Solution**: Use external secrets for Swarm/production

```bash
# Create secrets in Swarm
echo "db-password-value" | docker secret create prod_db_password -

# Update docker-compose.yml for production
secrets:
  db_password:
    external: true
    name: prod_db_password
```

---

## Migration Verification Report

After migration, generate a report:

```
🔐 Secrets Migration Report
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

✅ Migration Completed: 2025-10-20 14:30:00

Secrets Migrated (4):
✅ db_password (.env → Docker secrets)
✅ jwt_secret (.env → Docker secrets)
✅ api_key (docker-compose.yml → Docker secrets)
✅ smtp_password (.env → Docker secrets)

File Structure:
✅ ./secrets directory: 700 permissions
✅ All secret files: 600 permissions
✅ .gitignore updated
✅ .gitkeep present

Configuration Cleanup:
✅ .env: 0 secrets remaining
✅ docker-compose.yml: 0 secrets in environment
✅ .env.example: updated with migration notes

Docker Integration:
✅ 4 secrets defined in docker-compose.yml
✅ All services updated
✅ docker-entrypoint.sh created for app service

Validation:
✅ stack-validator: PASS
✅ All services running
✅ No secrets in git

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
✅ Migration successful - stack secured!

Next Steps:
1. Test all application functionality
2. Monitor logs for secret access issues
3. Plan secret rotation schedule (90 days)
4. Document secret setup in README.md
```

---

*Follow this guide to safely migrate secrets from insecure storage to Docker secrets.*
