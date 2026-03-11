---
name: "config-generator"
description: "Generates service-specific configuration files for GitLab stack projects in ./config directory, using .env as the primary configuration source. Creates nginx, PostgreSQL, Redis, and custom service configs with strict validation for secrets, paths, and Docker best practices. Use when setting up service configurations, creating config templates, or ensuring configs follow stack patterns."
---

# GitLab Stack Config Generator

Generates and manages service-specific configuration files for GitLab stack projects. Configurations follow proper patterns, use .env for all variables, and never contain secrets.

## When to Use

Activate when the user requests:
- Generate configuration files for services (nginx, PostgreSQL, Redis, etc.)
- Create config templates or set up `./config` directory structure
- Validate existing configuration files against stack patterns
- Set up project meta files (CLAUDE.md, .gitignore, .dockerignore)
- Sync .env and .env.example

## Core Rules

1. **.env is the configuration source** — all variables live in .env, not in config files
2. **No secrets in configs** — use secrets-manager for passwords, keys, tokens
3. **One directory per service** under `./config/service-name/`
4. **Flat structure** inside each service directory (subdirs only when logically needed)
5. **.env and .env.example must always match** in variable names
6. **All referenced paths must exist** — validate volume mounts and file references
7. **Use docker-validation skill** for Docker config validation
8. **Meta files required** — CLAUDE.md, .gitignore, .dockerignore in project root

## Configuration Workflow

### Phase 1: Understand Requirements

Determine (ask or infer):
- Which services need configuration?
- New configs or updating existing?
- Production, development, or both?
- Which template to use? (See [service-templates.md](service-templates.md))

Check current state:
1. Does `./config` directory exist?
2. Do `.env` and `.env.example` exist?
3. Do service directories already exist?
4. Are meta files present?

### Phase 2: Directory Structure Setup

```bash
mkdir -p ./config/nginx ./config/postgres ./config/redis
chmod 755 ./config ./config/*
```

Target structure:
```
./config/
├── nginx/
│   ├── nginx.conf
│   └── conf.d/
├── postgres/
│   ├── postgresql.conf
│   └── init.sql
├── redis/
│   └── redis.conf
└── app/
    └── settings.yml
```

### Phase 3: Meta Files Generation

Ensure these three files exist in the project root:

**CLAUDE.md** — Must include:
- Repository purpose and stack architecture overview
- Configuration patterns (variables in .env, configs in ./config, secrets in ./secrets)
- Rule: never mention "Claude" or "Claude Code" in commit messages
- Docker commands reference
- Directory structure overview
- Available skills list

**.gitignore** — Must exclude:
- `/secrets/` (keep `.gitkeep`), `.env` and `.env.local`, `/_temporary/`
- IDE files, OS files, logs, build artifacts, backup files

**.dockerignore** — Must exclude:
- `.git/`, `.env`, `secrets/`, `_temporary/`, `*.md`, IDE/OS files

### Phase 4: Service Configuration Generation

For each service, follow this pattern:

1. **Select template** from [service-templates.md](service-templates.md) or create custom
2. **Identify required .env variables** for the service
3. **Generate config file** using `${VAR_NAME}` placeholders — never hardcode values
4. **Update .env** with all required variables
5. **Sync .env.example** to match .env exactly
6. **Update docker-compose.yml** with volume mounts and environment references

Example nginx config generation:
```nginx
upstream app {
    server ${APP_HOST}:${APP_PORT};
}

server {
    listen ${NGINX_PORT};
    server_name ${NGINX_HOST};

    location / {
        proxy_pass http://app;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }
}
```

Corresponding .env additions:
```bash
NGINX_PORT=80
NGINX_HOST=localhost
APP_HOST=app
APP_PORT=8080
```

Corresponding docker-compose.yml entry:
```yaml
services:
  nginx:
    image: nginx:alpine
    ports:
      - "${NGINX_PORT}:80"
    volumes:
      - ./config/nginx/nginx.conf:/etc/nginx/nginx.conf:ro
    environment:
      NGINX_HOST: ${NGINX_HOST}
      APP_HOST: ${APP_HOST}
      APP_PORT: ${APP_PORT}
```

### Phase 5: Validation

Run all checks from [validation-rules.md](validation-rules.md):

1. **Syntax validation** — test each config type:
   ```bash
   # Nginx
   docker run --rm -v $(pwd)/config/nginx:/etc/nginx:ro nginx:alpine nginx -t
   # Redis
   docker run --rm -v $(pwd)/config/redis:/usr/local/etc/redis:ro redis:alpine redis-server --test-memory 1024
   ```

2. **Secret detection** — scan for leaked credentials:
   ```bash
   grep -r -iE "(password|secret|key|token|api_key)" ./config/
   ```
   If found, use secrets-manager skill immediately.

3. **Path validation** — confirm all referenced files and volume mounts exist

4. **.env sync check** — verify .env and .env.example have identical variable names:
   ```bash
   diff <(grep -E "^[A-Z_]+" .env | cut -d'=' -f1 | sort) \
        <(grep -E "^[A-Z_]+" .env.example | cut -d'=' -f1 | sort)
   ```

5. **Docker validation** — invoke docker-validation skill on docker-compose.yml

### Phase 6: Report and Next Steps

After generation, provide a summary report covering:
- Meta files created/updated
- Directory structure created
- For each service: config files generated, variables added, validation status
- Security validation results (no secrets in configs, .env sync status)
- Docker validation results
- Suggested next steps: `docker compose config` to validate, `docker compose up -d` to start

## Integration with Companion Skills

| Skill | When to Call |
|-------|-------------|
| **secrets-manager** | Secrets detected in configs; need to reference secrets; SSL certs needed |
| **docker-validation** | After any docker-compose.yml update; before completing generation |
| **stack-validator** | After generation complete; full stack validation |

## Reference Files

- **[service-templates.md](service-templates.md)** — Ready-to-use nginx, PostgreSQL, and Redis config templates with .env variables and docker-compose.yml integration
- **[validation-rules.md](validation-rules.md)** — Complete validation checklist and rules for secret detection, path validation, and .env synchronization
