---
name: "secrets-manager"
description: "Manages Docker secrets for GitLab stack projects, ensuring secrets are never in .env or docker-compose.yml, properly stored in ./secrets directory, and securely integrated with Docker secrets. Use when users need to create secrets, migrate from environment variables, validate secret configuration, audit secret usage, or ensure secrets are never committed to git."
---

# GitLab Stack Secrets Manager

## When to Use This Skill

Activate when the user requests any of:
- Create, rotate, or remove Docker secrets
- Migrate environment variables to Docker secrets
- Validate secret configuration and permissions
- Audit secret usage or detect leaks
- Fix secret-related security issues

## Security Rules

These rules are **never** violated. See [secrets-patterns.md](./secrets-patterns.md) for full rationale and patterns.

1. **No secrets in `.env`** or `docker-compose.yml` environment variables
2. All secrets stored in `./secrets/` directory (700 permissions)
3. Individual secret files have 600 permissions, non-root ownership
4. `./secrets/*` in `.gitignore`, never committed to git
5. Use Docker secrets mechanism exclusively
6. Never display actual secret values -- show `[REDACTED]`

## Workflow

### Step 1: Assess Current State

Determine the operation (create, migrate, validate, audit, rotate, remove) and gather context:

```bash
# Check project state
ls -ld ./secrets 2>/dev/null
ls -la ./secrets/ 2>/dev/null
cat .gitignore | grep secrets
```

```bash
# Scan for secrets in wrong places
grep -E "(PASSWORD|SECRET|KEY|TOKEN|API)" .env 2>/dev/null
grep -E "(PASSWORD|SECRET|KEY|TOKEN)" docker-compose.yml 2>/dev/null
```

```bash
# Check git safety
git status --porcelain | grep secrets/
git log --all --full-history -- ./secrets/ 2>/dev/null
```

**If secrets found in `.env` or `docker-compose.yml`**: flag as critical, proceed to [Migration](#step-3-migrate-secrets).

### Step 2: Create Secrets

**Prerequisites**: Ensure `./secrets` directory exists with correct permissions.

```bash
mkdir -p ./secrets && chmod 700 ./secrets
```

**Generate and store the secret** (no trailing newline):

```bash
# Alphanumeric (32 chars)
openssl rand -base64 32 | tr -d '/+=' | head -c 32 > ./secrets/secret_name

# Hex (64 chars)
openssl rand -hex 32 > ./secrets/secret_name

# Base64 (32 bytes)
openssl rand -base64 32 > ./secrets/secret_name

# UUID
uuidgen > ./secrets/secret_name
```

```bash
# Lock down permissions
chmod 600 ./secrets/secret_name
```

**Update `docker-compose.yml`**:

```yaml
# Top-level secrets section
secrets:
  secret_name:
    file: ./secrets/secret_name

# Add to service
services:
  myservice:
    secrets:
      - secret_name
```

**Update `.gitignore`**:

```gitignore
/secrets/
/secrets/*
!secrets/.gitkeep
```

**Error recovery**: If `chmod` fails (e.g., filesystem doesn't support POSIX permissions), verify the host filesystem type and document the limitation. On non-POSIX filesystems, rely on directory-level access controls instead.

### Step 3: Migrate Secrets

> For detailed migration scenarios and troubleshooting, see [migration-guide.md](./migration-guide.md).

**Identify** secrets to migrate:

```bash
grep -E "(PASSWORD|SECRET|KEY|TOKEN|API)" .env 2>/dev/null
```

**Confirm with user** which variables to migrate and their target secret names.

**For each secret**:

1. Extract value and create secret file:
   ```bash
   echo -n "$value" > ./secrets/secret_name
   chmod 600 ./secrets/secret_name
   ```

2. Add to `docker-compose.yml` secrets section (see Step 2).

3. Update service configuration. If the container supports `_FILE` suffix:
   ```yaml
   environment:
     DB_PASSWORD_FILE: /run/secrets/db_password
   ```

4. If no native secret support, generate a `docker-entrypoint.sh` (see [Entrypoint Generation](#entrypoint-generation) below).

5. Remove the secret from `.env` and/or `docker-compose.yml` environment.

**Error recovery**: If a service fails to start after migration:
- Check `docker compose logs <service>` for secret-loading errors
- Verify the secret file exists at `/run/secrets/<name>` inside the container: `docker compose exec <service> ls -la /run/secrets/`
- Confirm the service supports the `_FILE` suffix variant; if not, use the entrypoint approach
- Roll back by restoring the environment variable temporarily while debugging

### Step 4: Validate

Run this checklist after any create, migrate, or audit operation.

**Directory and permissions**:

```bash
# Directory exists with 700
[ -d ./secrets ] && stat -f "%OLp" ./secrets  # macOS
[ -d ./secrets ] && stat -c "%a" ./secrets     # Linux

# All files have 600 permissions
find ./secrets -type f ! -name .gitkeep -not -perm 600

# No root-owned files
find ./secrets -user root
```

**Docker Compose integration**:

```bash
# All referenced secret files exist
# (parse secrets: section and verify each file: path)
grep -A1 'file:' docker-compose.yml | grep -oP '\./secrets/\S+' | while read f; do
  [ -f "$f" ] && echo "OK: $f" || echo "MISSING: $f"
done
```

**Leak detection**:

```bash
# No secrets in .env
grep -cE "(PASSWORD|SECRET|KEY|TOKEN|API)" .env 2>/dev/null

# No secrets in compose environment sections
grep -A20 'environment:' docker-compose.yml | grep -iE "(password|secret|key|token)"

# .gitignore covers secrets
grep -q "secrets" .gitignore && echo "OK" || echo "MISSING"

# Nothing staged in git
git status --porcelain | grep secrets/
```

**Error recovery**: For each failing check, apply the corresponding fix:
- Missing directory -> `mkdir -p ./secrets && chmod 700 ./secrets`
- Wrong permissions -> `chmod 600 ./secrets/<file>` or `chmod 700 ./secrets`
- Root ownership -> `chown $(id -u):$(id -g) ./secrets/*`
- Missing `.gitignore` entry -> append `/secrets/` to `.gitignore`
- Secrets in `.env`/compose -> proceed to [Migration](#step-3-migrate-secrets)

Re-run validation after fixes until all checks pass.

### Step 5: Audit

**Secret inventory**:

```bash
find ./secrets -type f ! -name .gitkeep -exec ls -la {} \;
```

**Usage analysis**: For each secret file, verify it appears in the `docker-compose.yml` secrets section and is referenced by at least one service. Flag:
- Defined but unused secrets
- Secret files not in `docker-compose.yml`
- Referenced secrets with missing files

**Leak detection across the project**:

```bash
# Config files
grep -r "password\|secret\|key" ./config/ 2>/dev/null

# Git history
git log -p --all -S "PASSWORD" --diff-filter=A -- '*.env' '*.yml'
```

See [secrets-patterns.md](./secrets-patterns.md) for comprehensive detection patterns and common secret types.

## Entrypoint Generation

Required only when a container does not support native Docker secrets or the `_FILE` suffix convention.

**Containers with native support** (no entrypoint needed): PostgreSQL, MySQL, MariaDB, Redis, MongoDB.

For containers that only read environment variables, generate:

```bash
#!/bin/bash
set -e

load_secret() {
  local secret_name=$1
  local env_var=$2
  local secret_file="/run/secrets/${secret_name}"

  if [ -f "$secret_file" ]; then
    export "${env_var}=$(cat "$secret_file")"
  else
    echo "ERROR: Secret file $secret_file not found!" >&2
    exit 1
  fi
}

# Load required secrets
load_secret "db_password" "DB_PASSWORD"
load_secret "api_key" "API_KEY"

exec "$@"
```

```bash
chmod +x docker-entrypoint.sh
```

Update `docker-compose.yml`:

```yaml
services:
  myservice:
    entrypoint: /docker-entrypoint.sh
    command: ["original-command"]
    volumes:
      - ./docker-entrypoint.sh:/docker-entrypoint.sh:ro
    secrets:
      - db_password
      - api_key
```

**Error recovery**: If the entrypoint fails at runtime:
- Check the script is mounted correctly: `docker compose exec <service> cat /docker-entrypoint.sh`
- Verify secrets are mounted: `docker compose exec <service> ls /run/secrets/`
- Ensure the script has `+x` permission and uses LF line endings (not CRLF)

## Validation Report Format

After any operation, summarize results in this format:

```
Secrets Validation Report
=========================

Directory Structure
  [PASS] ./secrets exists with 700 permissions
  [PASS] Owned by user (not root)
  [PASS] ./secrets in .gitignore

Secret Files (N total)
  [PASS] db_password - 600 permissions, 32 bytes
  [FAIL] jwt_secret - 644 permissions (expected 600)

Docker Integration
  [PASS] N secrets defined in docker-compose.yml
  [PASS] All secret files exist

Leak Detection
  [PASS] No secrets in .env
  [FAIL] docker-compose.yml environment contains: JWT_SECRET
  [PASS] No secrets in git staging

Status: PASS / FAIL (N issues)

Required Actions:
1. Fix permissions on jwt_secret: chmod 600 ./secrets/jwt_secret
2. Migrate JWT_SECRET from compose environment to Docker secrets
```

## Companion Skills

- **stack-validator**: Calls this skill for secret validation; detects secrets in `.env` and compose files
- **stack-creator**: Creates `./secrets` directory, `.gitkeep`, `.gitignore` entries, and initial placeholders
- **config-generator**: Ensures configs reference secrets properly without embedding them
