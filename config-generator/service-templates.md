# Service Configuration Templates

This document provides ready-to-use configuration templates for common services in GitLab stack projects.

## Table of Contents
1. [Nginx Templates](#nginx-templates)
2. [PostgreSQL Templates](#postgresql-templates)
3. [Redis Templates](#redis-templates)
4. [Template Variables Reference](#template-variables-reference)

---

## Nginx Templates

### Template 1: Simple Reverse Proxy (Default)

**Use Case**: Basic reverse proxy to backend application

**Files**:
- `./config/nginx/nginx.conf`

**nginx.conf**:
```nginx
# Nginx Configuration - Simple Reverse Proxy
# Variables loaded from .env

events {
    worker_connections ${NGINX_WORKER_CONNECTIONS};
}

http {
    # Basic settings
    include /etc/nginx/mime.types;
    default_type application/octet-stream;

    # Logging
    access_log /var/log/nginx/access.log;
    error_log /var/log/nginx/error.log warn;

    # Performance
    sendfile on;
    keepalive_timeout 65;

    # Upstream backend
    upstream backend {
        server ${APP_BACKEND_HOST}:${APP_BACKEND_PORT};
    }

    server {
        listen ${NGINX_PORT};
        server_name ${NGINX_HOST};

        # Security headers
        add_header X-Frame-Options "SAMEORIGIN" always;
        add_header X-Content-Type-Options "nosniff" always;
        add_header X-XSS-Protection "1; mode=block" always;

        # Proxy to backend
        location / {
            proxy_pass http://backend;
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header X-Forwarded-Proto $scheme;
        }

        # Health check
        location /health {
            access_log off;
            return 200 "healthy\n";
            add_header Content-Type text/plain;
        }
    }
}
```

**.env variables**:
```bash
# Nginx - Simple Reverse Proxy
NGINX_PORT=80
NGINX_HOST=localhost
NGINX_WORKER_CONNECTIONS=1024
APP_BACKEND_HOST=app
APP_BACKEND_PORT=8080
```

**docker-compose.yml**:
```yaml
services:
  nginx:
    image: nginx:alpine
    container_name: ${COMPOSE_PROJECT_NAME}-nginx
    restart: unless-stopped
    ports:
      - "${NGINX_PORT}:80"
    volumes:
      - ./config/nginx/nginx.conf:/etc/nginx/nginx.conf:ro
    depends_on:
      - app
    networks:
      - app-network
```

---

### Template 2: SSL Termination

**Use Case**: HTTPS with SSL certificate from Docker secrets

**Files**:
- `./config/nginx/nginx.conf`

**nginx.conf**:
```nginx
# Nginx Configuration - SSL Termination
# SSL certificates from Docker secrets

events {
    worker_connections ${NGINX_WORKER_CONNECTIONS};
}

http {
    include /etc/nginx/mime.types;
    default_type application/octet-stream;

    access_log /var/log/nginx/access.log;
    error_log /var/log/nginx/error.log warn;

    sendfile on;
    keepalive_timeout 65;

    upstream backend {
        server ${APP_BACKEND_HOST}:${APP_BACKEND_PORT};
    }

    # HTTP server - redirect to HTTPS
    server {
        listen 80;
        server_name ${NGINX_HOST};

        location / {
            return 301 https://$server_name$request_uri;
        }
    }

    # HTTPS server
    server {
        listen 443 ssl http2;
        server_name ${NGINX_HOST};

        # SSL certificates from Docker secrets
        ssl_certificate /run/secrets/ssl_cert;
        ssl_certificate_key /run/secrets/ssl_key;

        # SSL settings
        ssl_protocols TLSv1.2 TLSv1.3;
        ssl_ciphers 'ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256';
        ssl_prefer_server_ciphers off;

        # Security headers
        add_header Strict-Transport-Security "max-age=31536000; includeSubDomains" always;
        add_header X-Frame-Options "SAMEORIGIN" always;
        add_header X-Content-Type-Options "nosniff" always;
        add_header X-XSS-Protection "1; mode=block" always;

        location / {
            proxy_pass http://backend;
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header X-Forwarded-Proto https;
        }
    }
}
```

**.env variables**:
```bash
# Nginx - SSL Termination
NGINX_HOST=example.com
NGINX_WORKER_CONNECTIONS=1024
APP_BACKEND_HOST=app
APP_BACKEND_PORT=8080
```

**Required secrets** (via secrets-manager):
- `ssl_cert` - SSL certificate
- `ssl_key` - SSL private key

**docker-compose.yml**:
```yaml
services:
  nginx:
    image: nginx:alpine
    container_name: ${COMPOSE_PROJECT_NAME}-nginx
    restart: unless-stopped
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - ./config/nginx/nginx.conf:/etc/nginx/nginx.conf:ro
    secrets:
      - ssl_cert
      - ssl_key
    depends_on:
      - app
    networks:
      - app-network

secrets:
  ssl_cert:
    file: ./secrets/ssl_cert.pem
  ssl_key:
    file: ./secrets/ssl_key.pem
```

---

### Template 3: Static Files + API Proxy

**Use Case**: Serve static frontend, proxy API requests to backend

**Files**:
- `./config/nginx/nginx.conf`

**nginx.conf**:
```nginx
# Nginx Configuration - Static + API Proxy

events {
    worker_connections ${NGINX_WORKER_CONNECTIONS};
}

http {
    include /etc/nginx/mime.types;
    default_type application/octet-stream;

    access_log /var/log/nginx/access.log;
    error_log /var/log/nginx/error.log warn;

    sendfile on;
    tcp_nopush on;
    keepalive_timeout 65;
    gzip on;
    gzip_vary on;
    gzip_types text/plain text/css application/json application/javascript text/xml application/xml;

    upstream api {
        server ${API_BACKEND_HOST}:${API_BACKEND_PORT};
    }

    server {
        listen ${NGINX_PORT};
        server_name ${NGINX_HOST};
        root ${NGINX_STATIC_ROOT};
        index index.html;

        # Static files
        location / {
            try_files $uri $uri/ /index.html;
            expires 1h;
            add_header Cache-Control "public, immutable";
        }

        # API proxy
        location /api/ {
            proxy_pass http://api/;
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header X-Forwarded-Proto $scheme;

            # No caching for API
            add_header Cache-Control "no-cache, no-store, must-revalidate";
        }

        # Assets with long cache
        location ~* \.(js|css|png|jpg|jpeg|gif|svg|ico|woff|woff2|ttf)$ {
            expires 1y;
            add_header Cache-Control "public, immutable";
        }
    }
}
```

**.env variables**:
```bash
# Nginx - Static + API Proxy
NGINX_PORT=80
NGINX_HOST=localhost
NGINX_WORKER_CONNECTIONS=1024
NGINX_STATIC_ROOT=/usr/share/nginx/html
API_BACKEND_HOST=api
API_BACKEND_PORT=3000
```

**docker-compose.yml**:
```yaml
services:
  nginx:
    image: nginx:alpine
    container_name: ${COMPOSE_PROJECT_NAME}-nginx
    restart: unless-stopped
    ports:
      - "${NGINX_PORT}:80"
    volumes:
      - ./config/nginx/nginx.conf:/etc/nginx/nginx.conf:ro
      - ./frontend/dist:/usr/share/nginx/html:ro
    depends_on:
      - api
    networks:
      - app-network
```

---

## PostgreSQL Templates

### Template 1: Basic (Default)

**Use Case**: Standard PostgreSQL for development

**Files**:
- `./config/postgres/postgresql.conf`
- `./config/postgres/init.sql`

**postgresql.conf**:
```conf
# PostgreSQL Configuration - Basic
# Variables from .env where applicable

# Connection settings
listen_addresses = '*'
max_connections = 100
shared_buffers = 128MB

# Logging
log_destination = 'stderr'
logging_collector = on
log_directory = 'log'
log_filename = 'postgresql-%Y-%m-%d.log'
log_statement = 'all'
log_duration = on

# Performance
effective_cache_size = 256MB
maintenance_work_mem = 64MB
checkpoint_completion_target = 0.9
wal_buffers = 16MB
default_statistics_target = 100
```

**init.sql**:
```sql
-- PostgreSQL Initialization Script
-- Creates database and user
-- Password loaded from Docker secrets

-- Note: POSTGRES_DB, POSTGRES_USER, POSTGRES_PASSWORD
-- are set via environment and Docker secrets

-- Create extensions if needed
CREATE EXTENSION IF NOT EXISTS "uuid-ossp";
CREATE EXTENSION IF NOT EXISTS "pg_trgm";

-- Grant privileges
GRANT ALL PRIVILEGES ON DATABASE ${POSTGRES_DB} TO ${POSTGRES_USER};
```

**.env variables**:
```bash
# PostgreSQL - Basic
POSTGRES_DB=myapp_db
POSTGRES_USER=myapp_user
# POSTGRES_PASSWORD in ./secrets/db_password
POSTGRES_PORT=5432
```

**docker-compose.yml**:
```yaml
services:
  postgres:
    image: postgres:16-alpine
    container_name: ${COMPOSE_PROJECT_NAME}-postgres
    restart: unless-stopped
    secrets:
      - db_password
    environment:
      POSTGRES_DB: ${POSTGRES_DB}
      POSTGRES_USER: ${POSTGRES_USER}
      POSTGRES_PASSWORD_FILE: /run/secrets/db_password
    volumes:
      - ./config/postgres/postgresql.conf:/etc/postgresql/postgresql.conf:ro
      - ./config/postgres/init.sql:/docker-entrypoint-initdb.d/init.sql:ro
      - postgres-data:/var/lib/postgresql/data
    ports:
      - "${POSTGRES_PORT}:5432"
    networks:
      - app-network

volumes:
  postgres-data:

secrets:
  db_password:
    file: ./secrets/db_password
```

---

### Template 2: Production

**Use Case**: Optimized PostgreSQL for production

**Files**:
- `./config/postgres/postgresql.conf`
- `./config/postgres/init.sql`

**postgresql.conf**:
```conf
# PostgreSQL Configuration - Production Optimized

# Connection settings
listen_addresses = '*'
max_connections = 200
superuser_reserved_connections = 3

# Memory settings (adjust based on available RAM)
shared_buffers = 256MB
effective_cache_size = 1GB
work_mem = 16MB
maintenance_work_mem = 128MB
wal_buffers = 16MB

# Checkpoints
checkpoint_timeout = 15min
checkpoint_completion_target = 0.9
max_wal_size = 2GB
min_wal_size = 1GB

# Performance
random_page_cost = 1.1
effective_io_concurrency = 200
default_statistics_target = 100

# Logging (production level)
log_destination = 'stderr'
logging_collector = on
log_directory = 'log'
log_filename = 'postgresql-%Y-%m-%d_%H%M%S.log'
log_rotation_age = 1d
log_rotation_size = 100MB
log_min_duration_statement = 1000  # Log slow queries > 1s
log_line_prefix = '%t [%p]: [%l-1] user=%u,db=%d,app=%a,client=%h '
log_checkpoints = on
log_connections = on
log_disconnections = on
log_lock_waits = on

# Autovacuum (important for production)
autovacuum = on
autovacuum_max_workers = 3
autovacuum_naptime = 10s

# Statement timeout (prevent runaway queries)
statement_timeout = 60000  # 60 seconds
```

**init.sql**:
```sql
-- PostgreSQL Production Initialization

CREATE EXTENSION IF NOT EXISTS "uuid-ossp";
CREATE EXTENSION IF NOT EXISTS "pg_trgm";
CREATE EXTENSION IF NOT EXISTS "btree_gin";
CREATE EXTENSION IF NOT EXISTS "btree_gist";

-- Performance monitoring
CREATE EXTENSION IF NOT EXISTS "pg_stat_statements";

-- Grant privileges
GRANT ALL PRIVILEGES ON DATABASE ${POSTGRES_DB} TO ${POSTGRES_USER};

-- Create schema
CREATE SCHEMA IF NOT EXISTS app AUTHORIZATION ${POSTGRES_USER};
```

**.env variables**:
```bash
# PostgreSQL - Production
POSTGRES_DB=prod_db
POSTGRES_USER=app_user
# POSTGRES_PASSWORD in ./secrets/db_password
POSTGRES_PORT=5432
POSTGRES_MAX_CONNECTIONS=200
```

---

## Redis Templates

### Template 1: Cache (Default)

**Use Case**: Redis as in-memory cache (no persistence)

**Files**:
- `./config/redis/redis.conf`

**redis.conf**:
```conf
# Redis Configuration - Cache (No Persistence)

# Network
bind 0.0.0.0
port 6379
protected-mode no

# Memory
maxmemory 256mb
maxmemory-policy allkeys-lru

# Persistence disabled for cache
save ""
appendonly no

# Performance
tcp-backlog 511
timeout 0
tcp-keepalive 300

# Logging
loglevel notice
logfile ""

# Snapshotting disabled
stop-writes-on-bgsave-error no
```

**.env variables**:
```bash
# Redis - Cache
REDIS_PORT=6379
REDIS_MAXMEMORY=256mb
REDIS_MAXMEMORY_POLICY=allkeys-lru
```

**docker-compose.yml**:
```yaml
services:
  redis:
    image: redis:7-alpine
    container_name: ${COMPOSE_PROJECT_NAME}-redis
    restart: unless-stopped
    command: redis-server /usr/local/etc/redis/redis.conf
    volumes:
      - ./config/redis/redis.conf:/usr/local/etc/redis/redis.conf:ro
    ports:
      - "${REDIS_PORT}:6379"
    networks:
      - app-network
```

---

### Template 2: Persistent

**Use Case**: Redis with data persistence

**Files**:
- `./config/redis/redis.conf`

**redis.conf**:
```conf
# Redis Configuration - Persistent

# Network
bind 0.0.0.0
port 6379
protected-mode no

# Memory
maxmemory 512mb
maxmemory-policy allkeys-lru

# RDB Persistence
save 900 1      # Save after 900 sec if 1 key changed
save 300 10     # Save after 300 sec if 10 keys changed
save 60 10000   # Save after 60 sec if 10000 keys changed
stop-writes-on-bgsave-error yes
rdbcompression yes
rdbchecksum yes
dbfilename dump.rdb
dir /data

# AOF Persistence
appendonly yes
appendfilename "appendonly.aof"
appendfsync everysec
no-appendfsync-on-rewrite no
auto-aof-rewrite-percentage 100
auto-aof-rewrite-min-size 64mb

# Performance
tcp-backlog 511
timeout 0
tcp-keepalive 300

# Logging
loglevel notice
logfile ""
```

**.env variables**:
```bash
# Redis - Persistent
REDIS_PORT=6379
REDIS_MAXMEMORY=512mb
REDIS_MAXMEMORY_POLICY=allkeys-lru
```

**docker-compose.yml**:
```yaml
services:
  redis:
    image: redis:7-alpine
    container_name: ${COMPOSE_PROJECT_NAME}-redis
    restart: unless-stopped
    command: redis-server /usr/local/etc/redis/redis.conf
    volumes:
      - ./config/redis/redis.conf:/usr/local/etc/redis/redis.conf:ro
      - redis-data:/data
    ports:
      - "${REDIS_PORT}:6379"
    networks:
      - app-network

volumes:
  redis-data:
```

---

### Template 3: Pub/Sub

**Use Case**: Redis for messaging/pub-sub (no persistence)

**Files**:
- `./config/redis/redis.conf`

**redis.conf**:
```conf
# Redis Configuration - Pub/Sub

# Network
bind 0.0.0.0
port 6379
protected-mode no

# Memory (higher for message buffers)
maxmemory 1gb
maxmemory-policy noeviction

# No persistence for pub/sub
save ""
appendonly no

# Pub/Sub specific
client-output-buffer-limit pubsub 32mb 8mb 60

# Performance
tcp-backlog 511
timeout 0
tcp-keepalive 60

# Logging
loglevel notice
logfile ""
```

**.env variables**:
```bash
# Redis - Pub/Sub
REDIS_PORT=6379
REDIS_MAXMEMORY=1gb
```

---

## Template Variables Reference

### Nginx Variables

| Variable | Default | Description |
|----------|---------|-------------|
| NGINX_PORT | 80 | Port nginx listens on |
| NGINX_HOST | localhost | Server name |
| NGINX_WORKER_CONNECTIONS | 1024 | Max connections per worker |
| NGINX_STATIC_ROOT | /usr/share/nginx/html | Static files root |
| APP_BACKEND_HOST | app | Backend service hostname |
| APP_BACKEND_PORT | 8080 | Backend service port |
| API_BACKEND_HOST | api | API service hostname |
| API_BACKEND_PORT | 3000 | API service port |

### PostgreSQL Variables

| Variable | Default | Description |
|----------|---------|-------------|
| POSTGRES_DB | myapp_db | Database name |
| POSTGRES_USER | myapp_user | Database user |
| POSTGRES_PORT | 5432 | Port PostgreSQL listens on |
| POSTGRES_MAX_CONNECTIONS | 100 | Maximum connections |

### Redis Variables

| Variable | Default | Description |
|----------|---------|-------------|
| REDIS_PORT | 6379 | Port Redis listens on |
| REDIS_MAXMEMORY | 256mb | Maximum memory |
| REDIS_MAXMEMORY_POLICY | allkeys-lru | Eviction policy |

---

*Use these templates as starting points and customize based on specific requirements.*
