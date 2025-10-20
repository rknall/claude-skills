# GitLab Stack Config Generator

> Generate service-specific configuration files for GitLab stack projects with .env as the primary configuration source

## Overview

The Config Generator skill creates service configurations for GitLab stack projects, ensuring all configs use .env variables, never contain secrets, and follow proper patterns. It also generates essential project meta files (CLAUDE.md, .gitignore, .dockerignore).

**Core Principle**: .env is the single source of configuration truth.

## Features

- **Service Templates**: Nginx, PostgreSQL, Redis (more services supported)
- **Meta Files**: CLAUDE.md, .gitignore, .dockerignore generation
- **.env Management**: All config from .env, strict .env/.env.example sync
- **Security**: No secrets in configs (uses secrets-manager)
- **Validation**: Syntax, paths, Docker configs (uses docker-validation)
- **Directory Structure**: Service-specific directories with flat configs

## When to Use

- Generate configuration files for services
- Set up new stack configurations
- Create project meta files (CLAUDE.md, .gitignore)
- Ensure .env and .env.example are synchronized
- Validate existing configurations

## Installation

```bash
/plugin marketplace add rknall/Skills
/plugin install config-generator
```

## Quick Start

### Generate Service Configs

```bash
# Generate nginx and PostgreSQL configs
claude "generate nginx and postgres configs"

# User will be prompted for template selection
```

### Generate Meta Files Only

```bash
# Create CLAUDE.md, .gitignore, .dockerignore
claude "generate project meta files"
```

## Available Templates

### Nginx
1. **Simple Reverse Proxy** (default) - Basic proxy to backend
2. **SSL Termination** - HTTPS with certificates from Docker secrets
3. **Static + API Proxy** - Serve static files + proxy API
4. **Custom** - User-specified requirements

### PostgreSQL
1. **Basic** (default) - Standard settings for development
2. **Production** - Optimized for production with monitoring
3. **With Extensions** - PostGIS, UUID, etc.
4. **Custom** - User-specified

### Redis
1. **Cache** (default) - No persistence, memory-only
2. **Persistent** - RDB + AOF persistence
3. **Pub/Sub** - Optimized for messaging
4. **Custom** - User-specified

## Configuration Principles

### 1. .env as Configuration Source

All configuration variables in .env:

```bash
# .env
NGINX_PORT=80
NGINX_HOST=localhost
POSTGRES_DB=myapp_db
REDIS_MAXMEMORY=256mb
```

### 2. No Secrets in Configs

Secrets ONLY in ./secrets via Docker secrets:

```yaml
# ❌ BAD - Secret in config
password: supersecret123

# ✅ GOOD - Reference to Docker secret
# Password loaded from /run/secrets/db_password
```

### 3. .env and .env.example Sync

**CRITICAL**: Must always match:

```bash
# Both files must have identical variable names
# Values in .env.example can be defaults/examples
```

### 4. Service Directory Structure

```
./config/
├── nginx/
│   └── nginx.conf         # Flat inside service dir
├── postgres/
│   ├── postgresql.conf
│   └── init.sql
└── redis/
    └── redis.conf
```

## Example Usage

### Complete Stack Setup

```bash
claude "Set up config for nginx, postgres, and redis"
```

**Workflow**:
1. Checks current state
2. Creates ./config/ directories
3. Generates meta files (CLAUDE.md, .gitignore, .dockerignore)
4. Asks for template selection
5. Generates configs with .env variables
6. Syncs .env.example
7. Updates docker-compose.yml
8. Validates (secrets, paths, Docker)
9. Provides comprehensive report

### Update Existing Config

```bash
claude "Update nginx config to use SSL"
```

## Validation

### Automatic Checks

1. **Secret Detection** (CRITICAL)
   - Scans all configs for secret patterns
   - Uses secrets-manager if found

2. **.env Sync** (CRITICAL)
   - Ensures .env and .env.example match
   - Reports any mismatches

3. **Path Validation**
   - All referenced paths exist
   - Volume mounts are valid

4. **Docker Validation**
   - Uses docker-validation skill
   - Addresses all findings

5. **Syntax Validation**
   - Nginx: `nginx -t`
   - PostgreSQL: SQL syntax
   - Redis: Config test

## Meta Files Generated

### CLAUDE.md

**CRITICAL CONTENT**:
- "NEVER mention Claude in commit messages"
- Stack architecture overview
- Configuration patterns
- Secrets management rules
- Directory structure
- Available skills

### .gitignore

Excludes:
- `/secrets/*` (except .gitkeep)
- `.env`
- `/_temporary/*`
- IDE files, OS files
- Logs, backups

### .dockerignore

Excludes from Docker builds:
- `.git/`, `.env`, `secrets/`
- Documentation (`*.md`)
- IDE files
- Logs

## Integration with Companion Skills

### secrets-manager
Called when:
- Secrets detected in configs
- SSL certificates needed
- Any secret-related operations

### docker-validation
Called for:
- docker-compose.yml validation (ALWAYS)
- Dockerfile validation if present

### stack-validator
Used for:
- Complete stack validation after generation
- Ensure all patterns followed

## Common Workflows

### New Stack Setup

```bash
# 1. Generate configs
claude "generate nginx, postgres, redis configs with SSL for nginx"

# 2. Review generated files
ls -la config/
cat .env
cat CLAUDE.md

# 3. Validate
docker compose config

# 4. Start
docker compose up -d
```

### Add Service to Existing Stack

```bash
claude "add redis config to my stack"
```

## Troubleshooting

### Issue: .env and .env.example out of sync

**Fix**:
```bash
claude "sync .env.example with .env"
```

### Issue: Secret detected in config

**Response**:
```
❌ CRITICAL: Secret detected in ./config/nginx/nginx.conf
Line 15: password=abc123

Action: Use secrets-manager to move to Docker secrets
```

### Issue: Docker validation fails

**Response**: Address docker-validation findings before completing

## Reference Documentation

- [SKILL.md](SKILL.md) - Complete workflow and validation
- [service-templates.md](service-templates.md) - All templates with examples
- [validation-rules.md](validation-rules.md) - Validation rules reference

## Version History

### v1.0.0 (2025-10-20)
- Initial release
- Nginx, PostgreSQL, Redis templates
- Meta files generation (CLAUDE.md, .gitignore, .dockerignore)
- Strict secret and .env validation
- Integration with secrets-manager and docker-validation

---

**Generate secure, validated service configurations following GitLab stack patterns.**
