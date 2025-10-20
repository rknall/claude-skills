# GitLab Stack Secrets Manager

> Secure Docker secrets management for GitLab stack projects - ensures secrets are never in .env or docker-compose.yml

## Overview

The GitLab Stack Secrets Manager skill helps you manage Docker secrets securely, ensuring secrets are never exposed in configuration files or version control. It focuses on migrating secrets from insecure locations (.env, docker-compose.yml environment variables) to Docker secrets.

**Core Principle**: Secrets MUST NEVER be in .env or docker-compose.yml environment variables.

## Features

- **Secret Creation**: Generate secure random secrets with proper permissions
- **Migration**: Move secrets from .env/docker-compose.yml to Docker secrets
- **Validation**: Detect secrets in wrong locations (CRITICAL security issue)
- **Auditing**: Find unused secrets, detect leaks, check permissions
- **Git Protection**: Ensure secrets never committed to version control
- **docker-entrypoint.sh Generation**: For containers without native secret support

## When to Use

Use this skill when you need to:
- Create new Docker secrets
- Migrate environment variables to Docker secrets
- Fix "secrets in .env" or "secrets in docker-compose.yml" issues
- Validate secret configuration and permissions
- Audit secret usage and detect leaks
- Generate secure random passwords/keys
- Rotate existing secrets
- Ensure secrets aren't in git

## Installation

```bash
# Install the skill marketplace
/plugin marketplace add rknall/Skills

# Install the secrets-manager skill
/plugin install secrets-manager
```

## Quick Start

### Critical Security Fix: Secrets in .env

If stack-validator found secrets in your .env file:

```bash
# Migrate all secrets from .env to Docker secrets
claude "migrate secrets from .env to Docker secrets"
```

This will:
1. Extract secret values from .env
2. Create ./secrets/secret_name files with proper permissions
3. Update docker-compose.yml to use Docker secrets
4. Remove secrets from .env
5. Verify the migration

### Create a New Secret

```bash
# Generate a secure random database password
claude "create a new secret db_password"

# Generate an API key
claude "create secret api_key --generate"
```

### Validate Secrets

```bash
# Check all secrets for issues
claude "validate secrets"
```

## Critical Security Rules

**These rules are NEVER violated**:

1. ❌ **NO secrets in .env** - Environment file must not contain secrets
2. ❌ **NO secrets in docker-compose.yml** - No plaintext in environment variables
3. ✅ **All secrets in ./secrets/** - With 700 directory permissions
4. ✅ **Secret files: 600 permissions** - Owner read/write only
5. ✅ **./secrets/* in .gitignore** - Never commit secrets
6. ✅ **Use Docker secrets only** - Native Docker secrets mechanism

## How Secrets Should Be Stored

### Correct Structure

```
./secrets/
├── .gitkeep              # Only file in git
├── db_password           # 600 permissions
├── api_key               # 600 permissions
└── jwt_secret            # 600 permissions

docker-compose.yml:
  secrets:
    db_password:
      file: ./secrets/db_password

  services:
    app:
      secrets:
        - db_password
      # Secret available at /run/secrets/db_password
```

### Incorrect (Security Risk)

```bash
# ❌ .env file
DB_PASSWORD=supersecret123        # CRITICAL SECURITY RISK!

# ❌ docker-compose.yml
services:
  app:
    environment:
      DB_PASSWORD: supersecret123  # CRITICAL SECURITY RISK!
```

## Common Use Cases

### 1. Migrate Secrets from .env

**Before** (.env):
```bash
DB_PASSWORD=mysecretpass
API_KEY=sk_live_abc123
```

**Command**:
```bash
claude "migrate DB_PASSWORD and API_KEY from .env to Docker secrets"
```

**After**:
- Secrets in ./secrets/db_password and ./secrets/api_key
- Updated docker-compose.yml to use Docker secrets
- Removed from .env

### 2. Fix Secrets in docker-compose.yml

**Before** (docker-compose.yml):
```yaml
services:
  app:
    environment:
      API_KEY: sk_live_hardcoded
```

**Command**:
```bash
claude "migrate API_KEY from docker-compose.yml to Docker secrets"
```

**After**:
```yaml
secrets:
  api_key:
    file: ./secrets/api_key

services:
  app:
    secrets:
      - api_key
```

### 3. Create New Random Secret

```bash
# Database password
claude "create db_password secret with 32 random characters"

# API key (hex format)
claude "generate api_key secret in hex format"

# JWT secret (base64)
claude "create jwt_secret as base64 encoded"
```

### 4. Validate All Secrets

```bash
claude "validate secrets configuration"
```

**Example Report**:
```
🔐 Secrets Validation Report
━━━━━━━━━━━━━━━━━━━━━━━━━━━

📁 Directory: ✅ 700 permissions
📄 Files: ✅ All 600 permissions

❌ CRITICAL SECURITY ISSUES
  ❌ .env contains secrets:
     * DB_PASSWORD
     * API_KEY
  ❌ docker-compose.yml has secrets in environment

🔧 IMMEDIATE ACTION REQUIRED
  Migrate secrets to Docker secrets now!
```

### 5. Audit Secrets

```bash
# Find unused secrets
claude "find unused secrets"

# Check for leaks
claude "check for secret leaks"

# List all secrets
claude "list all secrets with details"
```

## What Gets Validated

### Secret Detection Patterns

The skill detects these as secrets:

**Variable Names**:
- *PASSWORD*
- *SECRET*
- *KEY*
- *TOKEN*
- *API*
- *AUTH*
- *CREDENTIAL*

**Value Patterns**:
- Long random strings (40+ chars)
- Base64-encoded values
- Hex strings (64+ chars)
- JWT tokens
- API key formats (sk_live_*, pk_live_*)

### Validation Checks

1. **Directory Structure**
   - ./secrets exists with 700 permissions
   - Owned by current user (not root)
   - In .gitignore

2. **Secret Files**
   - 600 permissions (owner read/write only)
   - Not empty
   - Not root-owned

3. **.env File** (CRITICAL)
   - NO secrets detected
   - Only configuration values

4. **docker-compose.yml** (CRITICAL)
   - NO secrets in environment variables
   - All secrets in top-level secrets section
   - Services use secrets: key

5. **Git Safety**
   - ./secrets/* in .gitignore
   - No secrets in git history
   - No secrets staged for commit

## Migration Workflow

### Automatic Migration

When secrets detected in .env or docker-compose.yml:

1. **Detection**: Scans for secret patterns
2. **Extraction**: Saves values to ./secrets/ files
3. **Permissions**: Sets 700/600 on directory/files
4. **docker-compose.yml Update**: Adds secrets section
5. **Service Update**: Updates services to use secrets
6. **Cleanup**: Removes secrets from .env/compose
7. **Verification**: Validates migration success

### Manual Steps (if needed)

```bash
# 1. Create secret file
echo -n "secret-value" > ./secrets/db_password
chmod 600 ./secrets/db_password

# 2. Add to docker-compose.yml
# See examples in secrets-patterns.md

# 3. Remove from .env
sed -i '/DB_PASSWORD=/d' .env

# 4. Test
docker compose up -d
docker compose logs
```

## Integration with Companion Skills

### stack-validator
Automatically calls secrets-manager when:
- Secrets detected in .env
- Secrets found in docker-compose.yml environment
- Permission issues on ./secrets

### stack-creator
Creates proper secret structure:
- ./secrets directory with 700 permissions
- .gitkeep file
- .gitignore configuration

### config-generator
Ensures configs don't contain secrets, references secrets properly

## docker-entrypoint.sh Pattern

For containers that don't support native Docker secrets:

**When needed**:
- Container expects environment variables only
- No `_FILE` suffix support

**Example**:
```bash
#!/bin/bash
set -e

# Load secrets into environment
export DB_PASSWORD=$(cat /run/secrets/db_password)
export API_KEY=$(cat /run/secrets/api_key)

exec "$@"
```

**Containers with native support** (don't need entrypoint):
- PostgreSQL: `POSTGRES_PASSWORD_FILE`
- MySQL/MariaDB: `MYSQL_PASSWORD_FILE`
- Most modern containers

## Security Best Practices

### File Permissions

```bash
# Directory
chmod 700 ./secrets/          # drwx------

# Files
chmod 600 ./secrets/*         # -rw-------

# Ownership
chown $(id -u):$(id -g) ./secrets/*
```

### .gitignore

```gitignore
# Secrets - NEVER commit
/secrets/
/secrets/*
!secrets/.gitkeep

# Environment files
.env
.env.local

# Backups
*.old
*.backup
```

### Secret Generation

```bash
# Strong random passwords (32 chars)
openssl rand -base64 32 | tr -d '/+=' | head -c 32

# Hex keys (64 chars)
openssl rand -hex 32

# UUID
uuidgen
```

### Secret Rotation

```bash
# Backup old secret
cp ./secrets/api_key ./secrets/api_key.old

# Generate new
openssl rand -hex 32 > ./secrets/api_key

# Test
docker compose restart app

# Remove old after verification
rm ./secrets/api_key.old
```

## Troubleshooting

### Issue: Secrets detected in .env

**Fix**:
```bash
claude "migrate all secrets from .env to Docker secrets"
```

### Issue: Permission denied accessing secret

**Fix**:
```bash
chmod 700 ./secrets
chmod 600 ./secrets/*
```

### Issue: Secret file owned by root

**Fix**:
```bash
sudo chown $(id -u):$(id -g) ./secrets/*
```

### Issue: Service can't read secret

**Check**:
1. Secret defined in docker-compose.yml?
2. Service lists secret in `secrets:` key?
3. File exists in ./secrets/?
4. Proper permissions?

```bash
# Verify
docker compose exec app ls -la /run/secrets/
```

## Reference Documentation

- [SKILL.md](SKILL.md) - Complete skill workflow and validation process
- [secrets-patterns.md](secrets-patterns.md) - Security patterns and best practices
- [migration-guide.md](migration-guide.md) - Step-by-step migration scenarios

## Version History

### v1.0.0 (2025-10-20)
- Initial release
- Secret creation and management
- Migration from .env and docker-compose.yml
- Comprehensive validation and auditing
- docker-entrypoint.sh generation
- Git protection and leak detection
- Integration with stack-validator

## Contributing

Found an issue? This skill is part of the rknall-custom-skills marketplace.

## License

Part of the rknall-custom-skills marketplace for Claude Code.

---

**Secure your secrets - never expose them in configuration files or version control.**
