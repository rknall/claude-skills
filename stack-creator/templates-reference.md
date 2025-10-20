# Stack Templates Reference

This document provides ready-to-use templates for common stack configurations.

## Table of Contents

1. [docker-compose.yml Templates](#docker-composeyml-templates)
2. [.env.example Templates](#envexample-templates)
3. [Service Configuration Templates](#service-configuration-templates)
4. [Documentation Templates](#documentation-templates)
5. [Git Configuration Templates](#git-configuration-templates)

---

## docker-compose.yml Templates

### Minimal Stack (Development)

```yaml
services:
  app:
    image: myapp:latest
    container_name: ${PROJECT_NAME:-app}_main
    restart: unless-stopped
    environment:
      - NODE_ENV=development
    ports:
      - "${APP_PORT:-3000}:3000"
    networks:
      - app-network

networks:
  app-network:
    driver: bridge
```

### Web Stack (nginx + Application)

```yaml
services:
  nginx:
    image: nginx:alpine
    container_name: ${PROJECT_NAME:-app}_nginx
    restart: unless-stopped
    ports:
      - "${NGINX_HTTP_PORT:-80}:80"
      - "${NGINX_HTTPS_PORT:-443}:443"
    volumes:
      - ./config/nginx:/etc/nginx/conf.d:ro
      - ./ssl:/etc/nginx/ssl:ro
    networks:
      - app-network
    depends_on:
      - app

  app:
    image: myapp:latest
    container_name: ${PROJECT_NAME:-app}_main
    restart: unless-stopped
    environment:
      - NODE_ENV=production
    networks:
      - app-network

networks:
  app-network:
    driver: bridge
```

### Full Stack (nginx + App + PostgreSQL + Redis)

```yaml
services:
  nginx:
    image: nginx:alpine
    container_name: ${PROJECT_NAME:-app}_nginx
    restart: unless-stopped
    ports:
      - "${NGINX_HTTP_PORT:-80}:80"
      - "${NGINX_HTTPS_PORT:-443}:443"
    volumes:
      - ./config/nginx:/etc/nginx/conf.d:ro
    networks:
      - app-network
    depends_on:
      - app

  app:
    image: myapp:latest
    container_name: ${PROJECT_NAME:-app}_main
    restart: unless-stopped
    environment:
      - NODE_ENV=production
      - DB_HOST=postgres
      - DB_PORT=5432
      - DB_NAME=${POSTGRES_DB}
      - REDIS_HOST=redis
      - REDIS_PORT=6379
    secrets:
      - db_password
      - redis_password
    networks:
      - app-network
    depends_on:
      - postgres
      - redis

  postgres:
    image: postgres:16-alpine
    container_name: ${PROJECT_NAME:-app}_postgres
    restart: unless-stopped
    environment:
      - POSTGRES_DB=${POSTGRES_DB}
      - POSTGRES_USER=${POSTGRES_USER}
    secrets:
      - source: db_password
        target: /run/secrets/db_password
    volumes:
      - ./config/postgres:/docker-entrypoint-initdb.d:ro
      - postgres_data:/var/lib/postgresql/data
    networks:
      - app-network
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U ${POSTGRES_USER}"]
      interval: 10s
      timeout: 5s
      retries: 5

  redis:
    image: redis:7-alpine
    container_name: ${PROJECT_NAME:-app}_redis
    restart: unless-stopped
    command: ["redis-server", "/usr/local/etc/redis/redis.conf"]
    volumes:
      - ./config/redis/redis.conf:/usr/local/etc/redis/redis.conf:ro
      - redis_data:/data
    networks:
      - app-network
    healthcheck:
      test: ["CMD", "redis-cli", "ping"]
      interval: 10s
      timeout: 5s
      retries: 5

networks:
  app-network:
    driver: bridge

volumes:
  postgres_data:
    driver: local
  redis_data:
    driver: local

secrets:
  db_password:
    file: ./secrets/db_password
  redis_password:
    file: ./secrets/redis_password
```

### Stack with Docker Secrets (Production)

```yaml
services:
  app:
    image: myapp:latest
    container_name: ${PROJECT_NAME:-app}_main
    restart: unless-stopped
    environment:
      - NODE_ENV=production
      - DB_HOST=postgres
      - DB_USER=${DB_USER}
      # NO PASSWORDS IN ENVIRONMENT!
    secrets:
      - db_password
      - api_key
      - jwt_secret
    networks:
      - app-network

  postgres:
    image: postgres:16-alpine
    container_name: ${PROJECT_NAME:-app}_postgres
    restart: unless-stopped
    environment:
      - POSTGRES_DB=${POSTGRES_DB}
      - POSTGRES_USER=${POSTGRES_USER}
      # Password via Docker secret, not environment!
    secrets:
      - source: db_password
        target: /run/secrets/postgres-passwd
    volumes:
      - postgres_data:/var/lib/postgresql/data
    networks:
      - app-network

networks:
  app-network:
    driver: bridge

volumes:
  postgres_data:
    driver: local

secrets:
  db_password:
    file: ./secrets/db_password
  api_key:
    file: ./secrets/api_key
  jwt_secret:
    file: ./secrets/jwt_secret
```

---

## .env.example Templates

### Basic Application

```bash
# Project Configuration
PROJECT_NAME=myapp

# Application Settings
NODE_ENV=production
APP_PORT=3000

# IMPORTANT: Copy this file to .env and configure
# .env is gitignored and should contain actual values
```

### Web Stack with nginx

```bash
# Project Configuration
PROJECT_NAME=mywebapp

# nginx Configuration
NGINX_HTTP_PORT=80
NGINX_HTTPS_PORT=443

# Application Settings
NODE_ENV=production

# IMPORTANT: Copy this file to .env and configure for your environment
# .env is gitignored and should contain actual configuration
```

### Full Stack (nginx + App + PostgreSQL + Redis)

```bash
# Project Configuration
PROJECT_NAME=myapp

# nginx Configuration
NGINX_HTTP_PORT=80
NGINX_HTTPS_PORT=443

# Application Settings
NODE_ENV=production

# PostgreSQL Configuration
POSTGRES_DB=myapp_db
POSTGRES_USER=myapp_user
# NOTE: Password stored in Docker secret, not here!

# Redis Configuration
REDIS_PORT=6379
# NOTE: Password stored in Docker secret, not here!

# IMPORTANT SECURITY NOTES:
# 1. Copy this file to .env for actual configuration
# 2. NEVER put secrets/passwords here - use Docker secrets in ./secrets/
# 3. .env is gitignored and should NEVER be committed
# 4. Keep .env and .env.example keys synchronized
```

---

## Service Configuration Templates

### nginx - Simple Reverse Proxy

**File**: `config/nginx/default.conf`

```nginx
upstream app {
    server app:3000;
}

server {
    listen 80;
    server_name localhost;

    location / {
        proxy_pass http://app;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

### nginx - Production with SSL

**File**: `config/nginx/default.conf`

```nginx
upstream app {
    server app:3000;
    keepalive 64;
}

# Redirect HTTP to HTTPS
server {
    listen 80;
    server_name example.com www.example.com;
    return 301 https://$server_name$request_uri;
}

# HTTPS Server
server {
    listen 443 ssl http2;
    server_name example.com www.example.com;

    # SSL Configuration
    ssl_certificate /etc/nginx/ssl/cert.pem;
    ssl_certificate_key /etc/nginx/ssl/key.pem;
    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_ciphers HIGH:!aNULL:!MD5;
    ssl_prefer_server_ciphers on;

    # Security Headers
    add_header Strict-Transport-Security "max-age=31536000; includeSubDomains" always;
    add_header X-Frame-Options "SAMEORIGIN" always;
    add_header X-Content-Type-Options "nosniff" always;
    add_header X-XSS-Protection "1; mode=block" always;

    # Logging
    access_log /var/log/nginx/access.log;
    error_log /var/log/nginx/error.log;

    # Proxy Configuration
    location / {
        proxy_pass http://app;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;

        # Timeouts
        proxy_connect_timeout 60s;
        proxy_send_timeout 60s;
        proxy_read_timeout 60s;
    }

    # Static files caching
    location ~* \.(jpg|jpeg|png|gif|ico|css|js|woff|woff2)$ {
        proxy_pass http://app;
        expires 1y;
        add_header Cache-Control "public, immutable";
    }
}
```

### PostgreSQL - Initialization Script

**File**: `config/postgres/init.sql`

```sql
-- Initialize database

-- Create extensions
CREATE EXTENSION IF NOT EXISTS "uuid-ossp";
CREATE EXTENSION IF NOT EXISTS "pg_trgm";

-- Create schemas
CREATE SCHEMA IF NOT EXISTS app;

-- Create tables
CREATE TABLE IF NOT EXISTS app.users (
    id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
    email VARCHAR(255) UNIQUE NOT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Create indexes
CREATE INDEX idx_users_email ON app.users(email);

-- Grant permissions
GRANT ALL PRIVILEGES ON SCHEMA app TO ${POSTGRES_USER};
GRANT ALL PRIVILEGES ON ALL TABLES IN SCHEMA app TO ${POSTGRES_USER};
```

### Redis - Production Configuration

**File**: `config/redis/redis.conf`

```conf
# Redis Production Configuration

# Network
bind 0.0.0.0
protected-mode yes
port 6379

# General
daemonize no
supervised no
pidfile /var/run/redis_6379.pid
loglevel notice
logfile ""

# Snapshotting
save 900 1
save 300 10
save 60 10000
stop-writes-on-bgsave-error yes
rdbcompression yes
rdbchecksum yes
dbfilename dump.rdb
dir /data

# Replication
replica-serve-stale-data yes
replica-read-only yes

# Security
# requirepass will be set via environment variable
# Use Docker secrets for password

# Limits
maxmemory 256mb
maxmemory-policy allkeys-lru
maxclients 10000

# Append Only Mode
appendonly yes
appendfilename "appendonly.aof"
appendfsync everysec
no-appendfsync-on-rewrite no
auto-aof-rewrite-percentage 100
auto-aof-rewrite-min-size 64mb

# Slow Log
slowlog-log-slower-than 10000
slowlog-max-len 128
```

---

## Documentation Templates

### docs/setup.md

```markdown
# Setup Instructions

## Prerequisites

- Docker Engine 20.10+
- Docker Compose V2
- Git
- [Other prerequisites]

## Quick Start

\`\`\`bash
# Clone repository
git clone <repository-url>
cd <project-name>

# Copy environment template
cp .env.example .env

# Configure environment
nano .env

# Set up Docker secrets (if applicable)
# Follow instructions in ./secrets/README.md

# Install git hooks
./scripts/setup-hooks.sh

# Validate stack
./scripts/validate-stack.sh

# Start services
docker compose up -d

# Check status
docker compose ps
\`\`\`

## Detailed Setup

### 1. Environment Configuration

Edit `.env` with your settings:
\`\`\`bash
PROJECT_NAME=myapp
# Add other variables from .env.example
\`\`\`

### 2. Secrets Configuration

Create required secrets in `./secrets/`:
\`\`\`bash
# Generate secure random password
openssl rand -base64 32 > ./secrets/db_password

# Set proper permissions
chmod 600 ./secrets/*
\`\`\`

### 3. Validation

Always validate before deploying:
\`\`\`bash
./scripts/validate-stack.sh
\`\`\`

### 4. Deployment

\`\`\`bash
docker compose up -d
\`\`\`

## Troubleshooting

### Services Won't Start

\`\`\`bash
# Check logs
docker compose logs

# Check specific service
docker compose logs <service-name>
\`\`\`

### Validation Fails

\`\`\`bash
# Run individual validators
claude-code run stack-validator
claude-code run secrets-manager --validate
claude-code run docker-validation
\`\`\`

### Permission Issues

\`\`\`bash
# Fix file ownership
sudo chown -R $USER:$USER .

# Re-validate
./scripts/validate-stack.sh
\`\`\`
```

### docs/services.md

```markdown
# Services Documentation

## Service Overview

| Service | Port | Purpose | Configuration |
|---------|------|---------|---------------|
| nginx | 80, 443 | Web server & reverse proxy | ./config/nginx |
| app | 3000 (internal) | Application server | Environment variables |
| postgres | 5432 (internal) | Database | ./config/postgres |
| redis | 6379 (internal) | Cache & sessions | ./config/redis |

## Service Details

### nginx

**Image**: nginx:alpine
**Purpose**: Web server and reverse proxy
**Configuration**: ./config/nginx/default.conf
**Secrets**: None
**Volumes**:
- ./config/nginx:/etc/nginx/conf.d:ro
- ./ssl:/etc/nginx/ssl:ro (if using HTTPS)

**Health Check**: HTTP request to port 80

### Application

**Image**: myapp:latest
**Purpose**: Main application server
**Configuration**: Environment variables in .env
**Secrets**:
- db_password
- redis_password
- api_key
- jwt_secret

**Dependencies**:
- postgres (database)
- redis (cache)

### PostgreSQL

**Image**: postgres:16-alpine
**Purpose**: Primary database
**Configuration**: ./config/postgres/init.sql
**Secrets**:
- db_password

**Volumes**:
- postgres_data:/var/lib/postgresql/data (persistent)
- ./config/postgres:/docker-entrypoint-initdb.d:ro (init scripts)

**Health Check**: `pg_isready` command

**Backup**:
\`\`\`bash
docker compose exec postgres pg_dump -U ${POSTGRES_USER} ${POSTGRES_DB} > backup.sql
\`\`\`

### Redis

**Image**: redis:7-alpine
**Purpose**: Cache and session storage
**Configuration**: ./config/redis/redis.conf
**Secrets**:
- redis_password

**Volumes**:
- redis_data:/data (persistent)
- ./config/redis/redis.conf:/usr/local/etc/redis/redis.conf:ro

**Health Check**: `redis-cli ping`

**Backup**:
\`\`\`bash
docker compose exec redis redis-cli SAVE
docker compose cp redis:/data/dump.rdb ./backup/
\`\`\`

## Service Dependencies

\`\`\`
nginx → app → postgres
        ↓
       redis
\`\`\`

## Scaling

To scale the application:
\`\`\`bash
docker compose up -d --scale app=3
\`\`\`

Note: nginx configuration must support multiple backend servers.
```

### docs/decisions/0001-stack-architecture.md

```markdown
# 1. Stack Architecture

**Date**: YYYY-MM-DD
**Status**: Accepted
**Deciders**: [Names]

## Context

We need a consistent, maintainable approach for deploying our application stack.

## Decision

We will use GitLab Stack Management patterns:
1. All configuration in docker-compose.yml and ./config
2. All secrets in ./secrets and Docker secrets
3. docker-entrypoint.sh only when containers don't support native Docker secrets
4. No root-owned files
5. ./_temporary for transient files
6. Complete validation before deployment

## Consequences

**Positive**:
- Consistent structure across all stacks
- Automated validation prevents deployment issues
- Secure by default (secrets properly managed)
- Easy to maintain and update
- Clear separation of config and secrets

**Negative**:
- Initial setup requires more steps
- Must follow strict patterns
- Validation gates can slow rapid iteration
- Learning curve for team members

## Compliance

Stack creation enforces:
- stack-validator: Structure compliance
- secrets-manager: Secure secrets handling
- docker-validation: Docker best practices
- git hooks: Pre-commit validation

## Alternatives Considered

1. **Manual setup**: Rejected due to inconsistency
2. **.env for everything**: Rejected due to security concerns
3. **No validation**: Rejected due to quality issues

## Implementation

- Use stack-creator skill for all new projects
- Validate with ./scripts/validate-stack.sh
- Document all deviations in new ADRs
```

---

## Git Configuration Templates

### .gitignore

```gitignore
# Secrets - NEVER commit
secrets/*
!secrets/.gitkeep
!secrets/README.md
*.key
*.pem
*.crt
*.p12
*.pfx
id_rsa
id_ed25519

# Environment files
.env
.env.local
.env.*.local
!.env.example

# Temporary files
_temporary/
*.tmp
*.temp
.cache/
*.log

# Docker
.docker/

# IDE
.vscode/
.idea/
*.swp
*.swo
*~
.DS_Store
Thumbs.db

# Backup files
*.bak
*.backup
*.old

# Build artifacts
dist/
build/
node_modules/
```

### .dockerignore

```dockerignore
.git
.gitignore
.github
.gitlab-ci.yml
README.md
LICENSE
docs/
_temporary/
*.md
.env
.env.example
.env.local
secrets/
.vscode/
.idea/
*.swp
*.swo
*.log
*.tmp
node_modules/
.DS_Store
Thumbs.db
```

### CLAUDE.md Template

```markdown
# CLAUDE.md

This file provides guidance to Claude Code when working with this GitLab Stack project.

## Project Type

This is a GitLab Stack project following strict management patterns.

## Directory Structure

- `config/`: Service configurations (nginx, postgres, redis)
- `secrets/`: Docker secrets (NEVER commit actual secrets!)
- `_temporary/`: Temporary files (gitignored)
- `scripts/`: Validation and utility scripts
- `docs/`: Project documentation

## Required Skills

This project requires these Claude Code skills:
- **stack-validator**: Validate structure
- **secrets-manager**: Manage Docker secrets
- **docker-validation**: Validate Docker configs
- **config-generator**: Generate service configs

## Git Configuration

- **Branch**: main
- **Merge Strategy**: ff-only (fast-forward only)
- **Hooks**: Pre-commit validation enabled

## Validation Requirements

BEFORE any commit, ALL must pass:
1. stack-validator: NO issues
2. secrets-manager: Satisfied
3. docker-validation: NO issues
4. No root-owned files
5. No secrets in .env or docker-compose.yml environment

## Making Changes

### Adding a Service

1. Update docker-compose.yml
2. Use config-generator for configs
3. Use secrets-manager for secrets
4. Run ./scripts/validate-stack.sh
5. Fix ALL issues
6. Commit

### Modifying Configuration

1. Edit config files
2. Validate with docker-validation
3. Run ./scripts/validate-stack.sh
4. Fix issues
5. Commit

### Working with Secrets

1. Use secrets-manager for ALL secret operations
2. NEVER put secrets in .env
3. Use Docker secrets or ./secrets/
4. Validate before committing

## Important Rules

1. NEVER commit secrets
2. NEVER create root-owned files
3. NEVER skip validation without asking
4. NEVER use workarounds - ask user
5. ALWAYS validate before committing
6. ALWAYS document decisions
7. ALWAYS use ff-only merges
8. ALWAYS use main as branch name

## Troubleshooting

- Validation fails → Fix issues, don't skip
- Git conflicts → Use ff-only, ask user
- Permission issues → Check ownership, fix with chown
```

---

## Summary

All templates follow GitLab Stack Management principles:
- ✅ Secrets in Docker secrets, never in .env
- ✅ Configuration in ./config directory
- ✅ Validated before deployment
- ✅ Documented thoroughly
- ✅ Git properly configured

Use these templates as starting points and customize for your specific needs.
