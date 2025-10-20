---
name: "GitLab Stack Config Generator"
description: "Generates service-specific configuration files for GitLab stack projects in ./config directory, using .env as the primary configuration source. Creates nginx, PostgreSQL, Redis, and custom service configs with strict validation for secrets, paths, and Docker best practices. Use when setting up service configurations, creating config templates, or ensuring configs follow stack patterns."
---

# GitLab Stack Config Generator

This skill generates and manages service-specific configuration files for GitLab stack projects, ensuring configurations follow proper patterns, use .env for all variables, and never contain secrets.

## When to Use This Skill

Activate this skill when the user requests:
- Generate configuration files for services (nginx, PostgreSQL, Redis, etc.)
- Create config templates for new services
- Set up ./config directory structure
- Validate existing configuration files
- Ensure configs use .env variables correctly
- Check that configs don't contain secrets
- Set up project meta files (CLAUDE.md, .gitignore, .dockerignore)
- Sync .env and .env.example

## Core Configuration Principles

**CRITICAL RULES**:

1. **./env is Configuration Source**: All configuration variables in .env (NOT in config files)
2. **No Secrets in Configs**: NEVER put secrets in configuration files (use secrets-manager)
3. **Service Directories**: Each service gets `./config/service-name/` directory
4. **Flat Inside Service**: Config files flat inside service directory
5. **.env Sync**: .env and .env.example MUST always match
6. **Path Validation**: All referenced paths must exist
7. **Docker Validation**: Use docker-validation skill for Docker configs
8. **Project Meta Files**: CLAUDE.md, .gitignore, .dockerignore required

## Configuration Workflow

### Phase 1: Understanding Requirements

**Step 1: Determine What to Generate**

Ask the user (or infer):
- Which services need configuration?
- New configs or updating existing?
- Production, development, or both?
- Which templates to use (if any)?

**Step 2: Check Current State**

1. Does ./config directory exist?
2. Does .env exist?
3. Does .env.example exist?
4. Do service directories already exist?
5. Are meta files present (CLAUDE.md, .gitignore, .dockerignore)?

### Phase 2: Directory Structure Setup

**Step 1: Create ./config Directory**

```bash
mkdir -p ./config
chmod 755 ./config
```

**Step 2: Create Service Directories**

For each service (e.g., nginx, postgres, redis):

```bash
mkdir -p ./config/nginx
mkdir -p ./config/postgres
mkdir -p ./config/redis
chmod 755 ./config/*
```

**Directory Structure**:
```
./config/
├── nginx/
│   ├── nginx.conf
│   ├── ssl/
│   │   └── (SSL configs, not certificates)
│   └── conf.d/
│       └── default.conf
├── postgres/
│   ├── postgresql.conf
│   └── init.sql
├── redis/
│   └── redis.conf
└── app/
    └── settings.yml
```

**Principles**:
- One directory per service
- Flat structure inside each service directory
- Subdirectories only when logically needed (e.g., nginx/ssl/, nginx/conf.d/)

### Phase 3: Meta Files Generation

**CRITICAL**: Always ensure these files exist

**Step 1: Generate CLAUDE.md**

Create `CLAUDE.md` in project root:

```markdown
# CLAUDE.md

This file provides guidance when working with code in this repository.

## Repository Purpose

[Brief description of the stack project]

## Stack Architecture

This is a GitLab stack project following these principles:

- **Configuration**: All variables in .env, configs in ./config
- **Secrets**: All secrets in ./secrets via Docker secrets
- **Structure**: Standard directories (./config, ./secrets, ./_temporary)
- **Docker**: Modern Docker Compose (no version field)
- **Ownership**: No root-owned files

## Working with This Stack

### Configuration Files

All service configurations are in `./config/[service-name]/` directories.
Configuration values are loaded from `.env` file.

### Environment Variables

- `.env` contains all configuration (NOT secrets)
- `.env.example` must match `.env` exactly (critical requirement)
- Update both files when adding new variables

### Secrets Management

All secrets are managed via Docker secrets:
- Location: `./secrets/` directory
- Never in .env or docker-compose.yml
- Use secrets-manager skill for secret operations

### Docker Commands

```bash
# Start stack
docker compose up -d

# View logs
docker compose logs -f

# Stop stack
docker compose down
```

## Important Rules

### Git Commits

**CRITICAL**: When creating commit messages, NEVER mention "Claude" or "Claude Code" in the commit message.

Good commit messages:
- "Add nginx configuration for SSL termination"
- "Update PostgreSQL settings for production"
- "Fix Redis persistence configuration"

Bad commit messages:
- "Claude added nginx config"  ❌
- "Asked Claude to update settings"  ❌

### File Permissions

All files must be owned by the current user (not root):
```bash
# Check ownership
find . -user root -type f

# Fix if needed
sudo chown -R $(id -u):$(id -g) .
```

### Validation

Before deployment, always run:
```bash
# Validate entire stack
[validation command]

# Check secrets
[secrets validation command]
```

## Directory Structure

```
./
├── docker-compose.yml      # Main compose file (NO version field)
├── .env                    # Configuration variables
├── .env.example            # Template (must match .env)
├── CLAUDE.md               # This file
├── .gitignore              # Git exclusions
├── .dockerignore           # Docker build exclusions
├── config/                 # Service configurations
│   ├── nginx/
│   ├── postgres/
│   └── redis/
├── secrets/                # Docker secrets (NOT in git)
│   └── .gitkeep
└── _temporary/             # Transient files (NOT in git)
```

## Skills Available

- **stack-validator**: Validate entire stack before deployment
- **secrets-manager**: Manage Docker secrets securely
- **config-generator**: Generate/update service configurations
- **docker-validation**: Validate Docker configs

---

*Last updated: [date]*
```

**Step 2: Generate/Update .gitignore**

```gitignore
# Secrets - NEVER commit
/secrets/
/secrets/*
!secrets/.gitkeep

# Environment
.env
.env.local
.env.*.local

# Temporary
/_temporary/
/_temporary/*

# Docker
.dockerignore

# IDE
.vscode/
.idea/
*.swp
*.swo
*~

# OS
.DS_Store
Thumbs.db

# Logs
*.log
logs/

# Build artifacts
dist/
build/
node_modules/
vendor/

# Backup files
*.backup
*.old
*.bak
```

**Step 3: Generate .dockerignore**

```dockerignore
# Git
.git/
.gitignore

# Environment
.env
.env.*

# Secrets
secrets/

# Temporary
_temporary/

# Documentation
*.md
CLAUDE.md

# IDE
.vscode/
.idea/

# OS
.DS_Store
Thumbs.db

# Dependencies (if not needed in build)
node_modules/
vendor/

# Logs
*.log
logs/
```

### Phase 4: Service Configuration Generation

**Common Services**: nginx, PostgreSQL, Redis

For each service, follow this pattern:

**Step 1: Determine Configuration Needs**

Ask user:
- Use template or custom?
- Which features to enable?
- Production or development settings?

**Step 2: Identify Required .env Variables**

For each service, list all .env variables needed:

Example for nginx:
- NGINX_PORT
- NGINX_HOST
- NGINX_SSL_ENABLED
- APP_HOST
- APP_PORT

**Step 3: Generate Configuration File**

**CRITICAL**: Use environment variable placeholders (`${VAR_NAME}`)

Example nginx.conf:
```nginx
# Nginx Configuration
# Loads variables from .env

upstream app {
    server ${APP_HOST}:${APP_PORT};
}

server {
    listen ${NGINX_PORT};
    server_name ${NGINX_HOST};

    # SSL configuration
    # SSL certificates loaded from Docker secrets
    # ssl_certificate /run/secrets/ssl_cert;
    # ssl_certificate_key /run/secrets/ssl_key;

    location / {
        proxy_pass http://app;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }
}
```

**Step 4: Update .env**

Add all required variables:

```bash
# Nginx Configuration
NGINX_PORT=80
NGINX_HOST=localhost
NGINX_SSL_ENABLED=false
APP_HOST=app
APP_PORT=8080
```

**Step 5: Update .env.example**

**CRITICAL**: Must match .env exactly:

```bash
# Nginx Configuration
NGINX_PORT=80
NGINX_HOST=localhost
NGINX_SSL_ENABLED=false
APP_HOST=app
APP_PORT=8080
```

**Step 6: Update docker-compose.yml**

Add volume mount for config:

```yaml
services:
  nginx:
    image: nginx:alpine
    ports:
      - "${NGINX_PORT}:80"
    volumes:
      - ./config/nginx/nginx.conf:/etc/nginx/nginx.conf:ro
    environment:
      # Variables loaded from .env automatically
      NGINX_HOST: ${NGINX_HOST}
      APP_HOST: ${APP_HOST}
      APP_PORT: ${APP_PORT}
```

### Phase 5: Configuration Validation

**Step 1: Syntax Validation**

For each config file type:

**Nginx**:
```bash
# Test nginx config
docker run --rm -v $(pwd)/config/nginx:/etc/nginx:ro nginx:alpine nginx -t
```

**PostgreSQL**:
```bash
# Check SQL syntax
# Parse init.sql for syntax errors
```

**Redis**:
```bash
# Test redis config
docker run --rm -v $(pwd)/config/redis:/usr/local/etc/redis:ro redis:alpine redis-server --test-memory 1024
```

**Step 2: Secret Detection** (CRITICAL)

Scan all config files for secrets:

```bash
# Check for common secret patterns
grep -r -iE "(password|secret|key|token|api_key)" ./config/

# If found, this is CRITICAL SECURITY ISSUE
# Must use secrets-manager to fix
```

**Step 3: Path Validation**

Check that all referenced paths exist:

1. Config files reference other files?
2. Volume mounts in docker-compose.yml?
3. All paths exist?

**Step 4: .env Synchronization** (CRITICAL)

```bash
# Extract variable names from .env
env_vars=$(grep -E "^[A-Z_]+" .env | cut -d'=' -f1 | sort)

# Extract from .env.example
example_vars=$(grep -E "^[A-Z_]+" .env.example | cut -d'=' -f1 | sort)

# Compare
diff <(echo "$env_vars") <(echo "$example_vars")

# If any difference, this is CRITICAL ERROR
```

**Step 5: Docker Validation**

**CRITICAL**: Use docker-validation skill:

```
"Validate docker-compose.yml using docker-validation skill"
```

Address all findings before completing.

### Phase 6: Template Selection

**Step 1: Offer Templates**

For nginx, PostgreSQL, Redis, offer these options:

**Nginx Templates**:
1. **Simple Reverse Proxy** (default)
   - Basic proxy to backend service
   - No SSL
   - Minimal config

2. **SSL Termination**
   - HTTPS enabled
   - SSL certificates from Docker secrets
   - HTTP to HTTPS redirect

3. **Static + Proxy**
   - Serve static files
   - Proxy API requests
   - Caching headers

4. **Custom**
   - User provides requirements
   - Generate custom config

**PostgreSQL Templates**:
1. **Basic** (default)
   - Standard settings
   - Simple init.sql
   - Development friendly

2. **Production**
   - Optimized settings
   - Connection pooling
   - Performance tuning

3. **With Extensions**
   - PostGIS, UUID, etc.
   - Extension-specific config
   - Init scripts for extensions

4. **Custom**
   - User-specified

**Redis Templates**:
1. **Cache** (default)
   - No persistence
   - Memory-only
   - Fast

2. **Persistent**
   - RDB snapshots
   - AOF logging
   - Data durability

3. **Pub/Sub**
   - Optimized for messaging
   - No persistence
   - High throughput

4. **Custom**
   - User requirements

**Step 2: Generate from Template**

Based on user selection, generate appropriate config with:
- Required .env variables
- Proper docker-compose.yml integration
- README section documenting the config

### Phase 7: Validation Report

After generation, provide comprehensive report:

```
📝 Configuration Generation Report
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

✅ Meta Files
✅ CLAUDE.md created
✅ .gitignore updated
✅ .dockerignore created

✅ Directory Structure
✅ ./config/nginx created
✅ ./config/postgres created
✅ ./config/redis created

📄 Generated Configurations

Nginx (Simple Reverse Proxy):
✅ ./config/nginx/nginx.conf
✅ Variables added to .env (3)
✅ .env.example synced
✅ docker-compose.yml updated
✅ Syntax validation: PASS

PostgreSQL (Basic):
✅ ./config/postgres/postgresql.conf
✅ ./config/postgres/init.sql
✅ Variables added to .env (5)
✅ .env.example synced
✅ docker-compose.yml updated

Redis (Cache):
✅ ./config/redis/redis.conf
✅ Variables added to .env (2)
✅ .env.example synced
✅ docker-compose.yml updated

🔐 Security Validation
✅ No secrets in config files
✅ All secrets in ./secrets (via secrets-manager)
✅ Configs use .env variables only

✅ Path Validation
✅ All referenced paths exist
✅ Volume mounts valid

✅ .env Synchronization
✅ .env and .env.example match (10 variables)

🐳 Docker Validation
✅ docker-compose.yml syntax valid
✅ All volume mounts exist
✅ No deprecated syntax

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
✅ Configuration generation complete!

Next Steps:
1. Review generated configurations
2. Customize as needed
3. Run: docker compose config (validate)
4. Run: docker compose up -d
```

## Service-Specific Patterns

### Nginx Configuration

**Always include**:
- Environment variable placeholders
- Upstream backend reference
- Basic security headers
- Logging configuration

**Never include**:
- Hardcoded IPs or hosts
- SSL certificate content (use Docker secrets)
- Passwords or tokens

**Example variables**:
```bash
NGINX_PORT=80
NGINX_HOST=localhost
NGINX_WORKER_PROCESSES=auto
NGINX_WORKER_CONNECTIONS=1024
APP_BACKEND_HOST=app
APP_BACKEND_PORT=8080
```

### PostgreSQL Configuration

**Always include**:
- Init scripts for database/user creation
- Basic performance tuning
- Connection limits
- Logging settings

**Never include**:
- Passwords (use Docker secrets)
- Production database names in dev
- Hardcoded connection strings

**Example variables**:
```bash
POSTGRES_DB=myapp_db
POSTGRES_USER=myapp_user
# POSTGRES_PASSWORD in ./secrets/db_password
POSTGRES_MAX_CONNECTIONS=100
POSTGRES_SHARED_BUFFERS=256MB
```

### Redis Configuration

**Always include**:
- Persistence settings
- Memory limits
- Network bindings

**Never include**:
- Passwords (use Docker secrets if needed)
- Hardcoded master/slave IPs

**Example variables**:
```bash
REDIS_PORT=6379
REDIS_MAXMEMORY=256mb
REDIS_MAXMEMORY_POLICY=allkeys-lru
REDIS_SAVE_ENABLED=false
```

## Integration with Companion Skills

### secrets-manager
**When to call**:
- Secrets detected in config files → CRITICAL
- Need to reference secrets in configs
- SSL certificates or private keys needed

### docker-validation
**When to call**:
- After updating docker-compose.yml (ALWAYS)
- Before completing generation
- User requests validation

### stack-validator
**When to call**:
- After generation complete
- Validate entire stack
- Ensure all patterns followed

## Important Notes

- **No Secrets**: NEVER put secrets in config files
- **.env Source**: All configuration from .env
- **.env Sync**: .env and .env.example must match
- **Docker Validation**: Always use docker-validation skill
- **CLAUDE.md Rule**: Must specify no "Claude" in commit messages
- **Templates Optional**: User chooses defaults or custom
- **Path Validation**: All paths must exist

## Example Workflow

```
User: "Generate nginx and PostgreSQL configs"

1. Check current state
   - ./config missing → create it
   - .env exists
   - .env.example exists
   - CLAUDE.md missing → create it

2. Generate meta files
   - Create CLAUDE.md
   - Update .gitignore
   - Create .dockerignore

3. Ask user for templates
   "Which templates would you like?
    - Nginx: [Simple Reverse Proxy], SSL Termination, Static + Proxy, Custom
    - PostgreSQL: [Basic], Production, With Extensions, Custom"

4. User selects: Simple Reverse Proxy, Basic

5. Generate nginx config
   - Create ./config/nginx/nginx.conf
   - Add variables to .env
   - Sync .env.example
   - Update docker-compose.yml
   - Validate syntax

6. Generate PostgreSQL config
   - Create ./config/postgres/postgresql.conf
   - Create ./config/postgres/init.sql
   - Add variables to .env
   - Sync .env.example
   - Update docker-compose.yml

7. Run validations
   - Secret detection: PASS
   - Path validation: PASS
   - .env sync check: PASS
   - Docker validation (via docker-validation skill): PASS

8. Generate report
   - Show all created files
   - List .env variables added
   - Confirmation that validations passed

9. Next steps
   - Suggest testing: docker compose config
   - Recommend: docker compose up -d
```

---

*This skill generates service configurations following GitLab stack patterns with strict validation.*
