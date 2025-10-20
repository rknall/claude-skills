# Common Stack Validation Issues

This guide documents common issues found during stack validation, their causes, impacts, and remediation guidance.

## Table of Contents
1. [Critical Issues](#critical-issues)
2. [Security Issues](#security-issues)
3. [Configuration Issues](#configuration-issues)
4. [Structural Issues](#structural-issues)
5. [Docker Issues](#docker-issues)
6. [Ownership Issues](#ownership-issues)

---

## Critical Issues

### 1. Missing .env.example File

**Issue**: .env file exists but .env.example is missing

**Impact**:
- New developers don't know which environment variables are required
- No template for setting up new environments
- Difficult to understand configuration requirements
- Potential for missing critical variables in deployments

**Detection**:
```
❌ Environment Variables: CRITICAL
   - .env.example file not found
   - Required for documenting environment configuration
```

**Why This Matters**:
The .env.example file serves as documentation and a template for all required environment variables. Without it, setting up the stack in a new environment is error-prone.

**Remediation**:
Use stack-creator skill to generate .env.example from current .env file:
- `claude "Use stack-creator to add .env.example"`
- Or manually create .env.example with all variables from .env

---

### 2. .env and .env.example Out of Sync

**Issue**: Variables in .env don't match .env.example

**Impact**:
- Deployments may fail due to missing variables
- Documentation is inaccurate
- Team members may have different configurations
- Difficult to identify required vs optional variables

**Detection**:
```
❌ Environment Variables: CRITICAL
   - Variables in .env but NOT in .env.example:
     * API_TIMEOUT
     * MAX_CONNECTIONS
   - Variables in .env.example but NOT in .env:
     * CACHE_TTL
```

**Why This Matters**:
The two files MUST be synchronized. .env.example is the source of truth for required variables.

**Remediation**:
1. Manually sync the files - add missing variables
2. Use stack-creator to regenerate .env.example
3. Review if any variables should be removed

---

### 3. Secrets in .env File

**Issue**: Passwords, API keys, or tokens in .env

**Impact**:
- Security risk if .env is accidentally committed
- Secrets not properly isolated
- Violates security best practices
- Difficult to rotate secrets

**Detection**:
```
❌ Environment Variables: CRITICAL
   - Potential secrets detected in .env:
     * DB_PASSWORD (contains password pattern)
     * API_KEY (contains key pattern)
     * JWT_SECRET (contains secret pattern)
```

**Why This Matters**:
Secrets should NEVER be in .env files. They should be in ./secrets and managed via Docker secrets.

**Remediation**:
Use secrets-manager skill to properly configure secrets:
1. Move secrets to ./secrets directory
2. Update docker-compose.yml to use Docker secrets
3. Remove secrets from .env
4. Update applications to read from /run/secrets/

---

## Security Issues

### 4. ./secrets Directory Not in .gitignore

**Issue**: Secrets directory not excluded from git

**Impact**:
- **CRITICAL SECURITY RISK**: Secrets may be committed to git
- Secret exposure in version control
- Compliance violations
- Potential data breach

**Detection**:
```
❌ Directory Structure: CRITICAL
   - ./secrets not found in .gitignore
   - Risk of committing secrets to version control
```

**Why This Matters**:
Once secrets are committed to git, they remain in history forever, even after removal.

**Remediation**:
1. Add to .gitignore immediately:
   ```gitignore
   /secrets/
   /secrets/*
   ```
2. Check git status: `git status` - ensure no secrets staged
3. If secrets were committed, rotate all secrets immediately
4. Use BFG Repo-Cleaner to remove from git history (if needed)

---

### 5. Secrets in docker-compose.yml Environment Variables

**Issue**: Hardcoded secrets in environment variables

**Impact**:
- Secrets exposed in docker-compose.yml (may be in git)
- Violates security best practices
- Difficult to rotate secrets
- Potential compliance issues

**Detection**:
```
❌ Secrets Management: CRITICAL
   - Service 'app' has potential secrets in environment:
     * DB_PASSWORD=supersecret123 (hardcoded password)
   - Use Docker secrets instead
```

**Why This Matters**:
docker-compose.yml is often committed to git. Any secrets in it are exposed.

**Remediation**:
Use secrets-manager skill:
1. Move secrets to ./secrets/
2. Define in top-level secrets section
3. Reference via secrets key in services
4. Update app to read from /run/secrets/

---

### 6. Insecure ./secrets Directory Permissions

**Issue**: ./secrets directory readable by others (e.g., 755 instead of 700)

**Impact**:
- Other users on system can read secrets
- Security compliance violation
- Unnecessary exposure of sensitive data

**Detection**:
```
⚠️  Secrets Management: WARNING
   - ./secrets directory permissions: 755
   - Should be 700 (owner only)
```

**Why This Matters**:
Secrets should only be readable by the user running the stack.

**Remediation**:
```bash
chmod 700 ./secrets/
chmod 600 ./secrets/*
```

---

### 7. .env File Tracked in Git

**Issue**: .env file is committed to version control

**Impact**:
- Configuration values exposed in git
- May contain secrets (critical if so)
- Environment-specific config in wrong place
- Difficult to manage per-environment settings

**Detection**:
```
❌ Directory Structure: CRITICAL
   - .env is tracked by git (run: git ls-files .env)
   - Must be added to .gitignore
```

**Why This Matters**:
.env files often contain environment-specific and sensitive data.

**Remediation**:
1. Add .env to .gitignore
2. Remove from git: `git rm --cached .env`
3. Commit the removal
4. Review .env for any secrets - rotate if found

---

## Configuration Issues

### 8. Docker Compose Version Field Present

**Issue**: docker-compose.yml includes version field

**Impact**:
- Using deprecated syntax
- May cause warnings in modern Docker
- Not following current best practices

**Detection**:
```
⚠️  Docker Configuration: WARNING
   - docker-compose.yml contains 'version' field
   - Modern Compose files should omit version
```

**Why This Matters**:
Docker Compose Specification removed version field requirement. Modern files don't need it.

**Remediation**:
Manually remove the version line:
```yaml
# ❌ Remove this
version: '3.8'

# ✅ Start directly with services
services:
  app:
    ...
```

---

### 9. Secrets Defined but Files Missing

**Issue**: docker-compose.yml references secrets that don't exist

**Impact**:
- Stack will fail to start
- Docker will show "secret not found" errors
- Service initialization failures

**Detection**:
```
❌ Secrets Management: CRITICAL
   - Secret 'db_password' defined but file not found:
     Expected: ./secrets/db_password
   - Secret 'api_key' defined but file not found:
     Expected: ./secrets/api_key
```

**Why This Matters**:
Docker requires all referenced secret files to exist before starting services.

**Remediation**:
Use secrets-manager skill to create missing secrets:
1. Create secret files: `echo -n "value" > ./secrets/db_password`
2. Set proper permissions: `chmod 600 ./secrets/db_password`
3. Verify all secrets exist before starting stack

---

### 10. Undefined Secret References

**Issue**: Service references secret not defined in top-level secrets section

**Impact**:
- Docker Compose validation will fail
- Stack won't start
- Error messages: "service X references undefined secret"

**Detection**:
```
❌ Docker Configuration: CRITICAL
   - Service 'app' references undefined secret: jwt_secret
   - Secret must be defined in top-level 'secrets' section
```

**Why This Matters**:
All secrets used by services must be declared in the top-level secrets section.

**Remediation**:
Add secret definition to docker-compose.yml:
```yaml
secrets:
  jwt_secret:
    file: ./secrets/jwt_secret
```

---

## Structural Issues

### 11. Missing Required Directories

**Issue**: Required directories (./config, ./secrets, ./_temporary) don't exist

**Impact**:
- Stack may fail to start
- Volume mounts will fail
- Application errors
- Inconsistent structure

**Detection**:
```
❌ Directory Structure: CRITICAL
   - Required directory './secrets' not found
   - Required directory './_temporary' not found
```

**Why This Matters**:
Standard stack architecture requires these directories for proper organization.

**Remediation**:
Use stack-creator skill to create missing directories:
```bash
mkdir -p config secrets _temporary
chmod 700 secrets
```

---

### 12. Incomplete .gitignore

**Issue**: .gitignore doesn't exclude required paths

**Impact**:
- Risk of committing secrets or temporary files
- Large repository size
- Cluttered git status
- Security risks

**Detection**:
```
⚠️  Directory Structure: WARNING
   - .gitignore missing exclusion: /secrets/
   - .gitignore missing exclusion: /_temporary/
   - .gitignore missing exclusion: .env
```

**Why This Matters**:
Without proper exclusions, sensitive files may be accidentally committed.

**Remediation**:
Add to .gitignore:
```gitignore
# Secrets
/secrets/
/secrets/*

# Environment
.env

# Temporary
/_temporary/
/_temporary/*
```

---

### 13. ./_temporary Directory Not Empty

**Issue**: Temporary directory contains leftover files

**Impact**:
- Disk space waste
- Stale cached data
- Potential application issues
- Poor housekeeping

**Detection**:
```
⚠️  Temporary Directory: WARNING
   - ./_temporary is not empty:
     * cache/sessions/old_session.dat (modified 15 days ago)
     * uploads/temp_file.jpg (modified 3 days ago)
   - Should be cleaned after use
```

**Why This Matters**:
Temporary directories should be cleaned regularly to avoid accumulating stale data.

**Remediation**:
Manually clean the directory:
```bash
rm -rf ./_temporary/*
```

---

## Docker Issues

### 14. Invalid docker-compose.yml Syntax

**Issue**: YAML syntax errors in docker-compose.yml

**Impact**:
- Stack cannot start
- Docker Compose validation fails
- Difficult to troubleshoot
- Deployment failures

**Detection**:
```
❌ Docker Configuration: CRITICAL
   - docker-compose.yml syntax error at line 23
   - Error: mapping values are not allowed here
```

**Why This Matters**:
Invalid YAML prevents Docker Compose from parsing the file.

**Remediation**:
1. Fix YAML syntax errors
2. Validate with: `docker compose config`
3. Use YAML linter for complex files

---

### 15. Missing Service Dependencies

**Issue**: Services don't declare depends_on for required services

**Impact**:
- Services may start in wrong order
- Application errors on startup
- Race conditions
- Connection failures

**Detection**:
```
⚠️  Docker Configuration: WARNING
   - Service 'app' uses DB but doesn't declare depends_on: postgres
   - May cause startup ordering issues
```

**Why This Matters**:
Docker doesn't guarantee service start order without depends_on.

**Remediation**:
Add depends_on to service:
```yaml
services:
  app:
    depends_on:
      - postgres
      - redis
```

---

### 16. No Restart Policy

**Issue**: Services missing restart policy

**Impact**:
- Services don't recover from crashes
- Manual intervention needed
- Poor reliability
- Downtime

**Detection**:
```
⚠️  Docker Configuration: WARNING
   - Service 'app' has no restart policy
   - Recommend: restart: unless-stopped
```

**Why This Matters**:
Production services should automatically restart on failure.

**Remediation**:
Add restart policy:
```yaml
services:
  app:
    restart: unless-stopped
```

---

## Ownership Issues

### 17. Root-Owned Configuration Files

**Issue**: Files in ./config owned by root

**Impact**:
- Cannot modify config without sudo
- Inconsistent permissions
- Potential deployment issues
- Bad practice

**Detection**:
```
❌ File Ownership: CRITICAL
   - Root-owned files found:
     * ./config/nginx/nginx.conf (owner: root)
     * ./config/app/settings.yml (owner: root)
```

**Why This Matters**:
All stack files should be owned by the Docker user, not root.

**Remediation**:
Fix ownership:
```bash
sudo chown -R $(id -u):$(id -g) ./config/
```

To prevent: Ensure containers don't run as root, or don't mount config as writable.

---

### 18. Root-Owned Files in _temporary

**Issue**: Files in ./_temporary created by containers as root

**Impact**:
- Cannot clean directory without sudo
- Disk space issues
- Permission problems

**Detection**:
```
⚠️  File Ownership: WARNING
   - Root-owned files in ./_temporary:
     * ./_temporary/cache/app_cache.db (owner: root)
```

**Why This Matters**:
Temporary files should be cleanable by the user.

**Remediation**:
1. Fix current files: `sudo chown -R $(id -u):$(id -g) ./_temporary/`
2. Prevent future issues: Run containers as current user:
   ```yaml
   services:
     app:
       user: "${UID}:${GID}"
   ```

---

### 19. Root-Owned Secret Files

**Issue**: Files in ./secrets owned by root

**Impact**:
- Cannot update secrets without sudo
- Security risk (root has access)
- Operational difficulties

**Detection**:
```
❌ File Ownership: CRITICAL
   - Root-owned files in ./secrets:
     * ./secrets/db_password (owner: root)
```

**Why This Matters**:
Secret files should be owned by the Docker user with restricted permissions.

**Remediation**:
```bash
sudo chown $(id -u):$(id -g) ./secrets/*
chmod 600 ./secrets/*
```

---

## Script Issues

### 20. Unnecessary docker-entrypoint.sh

**Issue**: docker-entrypoint.sh exists but container supports native secrets

**Impact**:
- Unnecessary complexity
- Maintenance overhead
- Potential bugs
- Not following best practices

**Detection**:
```
⚠️  Scripts: WARNING
   - docker-entrypoint.sh found for service 'postgres'
   - PostgreSQL supports native Docker secrets via *_FILE variables
   - Consider removing custom entrypoint
```

**Why This Matters**:
Many modern containers support Docker secrets natively. Custom entrypoints add complexity.

**Remediation**:
1. Check if container supports *_FILE environment variables
2. If yes, remove docker-entrypoint.sh
3. Use native secret support:
   ```yaml
   environment:
     POSTGRES_PASSWORD_FILE: /run/secrets/db_password
   ```

---

### 21. docker-entrypoint.sh Not Executable

**Issue**: Script exists but doesn't have execute permissions

**Impact**:
- Container will fail to start
- Permission denied errors
- Deployment failures

**Detection**:
```
❌ Scripts: CRITICAL
   - docker-entrypoint.sh is not executable
   - Current permissions: -rw-r--r--
   - Required: -rwxr-xr-x
```

**Why This Matters**:
Docker needs execute permission to run entrypoint scripts.

**Remediation**:
```bash
chmod +x docker-entrypoint.sh
```

---

### 22. Hardcoded Secrets in Scripts

**Issue**: Secrets hardcoded in docker-entrypoint.sh or other scripts

**Impact**:
- **CRITICAL SECURITY RISK**
- Secrets exposed in git
- Difficult to rotate
- Compliance violations

**Detection**:
```
❌ Scripts: CRITICAL
   - Hardcoded secret detected in docker-entrypoint.sh:
     Line 15: export DB_PASSWORD="supersecret123"
   - Secrets must be read from /run/secrets/
```

**Why This Matters**:
Scripts are often committed to git. Hardcoded secrets = exposed secrets.

**Remediation**:
Read secrets from Docker secrets:
```bash
# ❌ BAD
export DB_PASSWORD="supersecret123"

# ✅ GOOD
export DB_PASSWORD=$(cat /run/secrets/db_password)
```

---

## Summary: Issue Severity Levels

### Critical (Must Fix Before Deployment)
- Missing .env.example
- .env/.env.example out of sync
- Secrets in .env
- ./secrets not in .gitignore
- Secrets in docker-compose.yml
- Missing secret files
- Root-owned files
- Invalid docker-compose.yml syntax

### High (Fix Soon)
- .env tracked in git
- Insecure secrets permissions
- Missing required directories
- Hardcoded secrets in scripts

### Medium (Should Fix)
- Docker Compose version field
- Missing restart policies
- Missing dependencies
- Incomplete .gitignore

### Low (Nice to Fix)
- ./_temporary not empty
- Unnecessary docker-entrypoint.sh
- Suboptimal configurations

---

## Quick Reference: Issue to Skill Mapping

| Issue Category | Recommended Skill |
|---------------|------------------|
| Missing .env.example | stack-creator |
| Directory structure | stack-creator |
| Secrets management | secrets-manager |
| Docker configuration | docker-validation (auto-used) |
| File ownership | Manual fix (chown) |
| .gitignore issues | stack-creator |
| .env sync | Manual + stack-creator |

---

*This guide helps identify and understand common stack validation issues for faster remediation.*
