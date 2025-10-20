---
name: "GitLab Stack Secrets Manager"
description: "Manages Docker secrets for GitLab stack projects, ensuring secrets are never in .env or docker-compose.yml, properly stored in ./secrets directory, and securely integrated with Docker secrets. Use when users need to create secrets, migrate from environment variables, validate secret configuration, audit secret usage, or ensure secrets are never committed to git."
---

# GitLab Stack Secrets Manager

This skill manages secrets for GitLab stack projects, ensuring secrets are stored securely, never exposed in configuration files, and properly integrated with Docker secrets.

## When to Use This Skill

Activate this skill when the user requests:
- Create or manage Docker secrets
- Migrate environment variables to Docker secrets
- Validate secret configuration and permissions
- Audit secret usage and detect leaks
- Ensure secrets aren't in .env or docker-compose.yml
- Check if secrets are exposed in git
- Generate secure random secrets
- Rotate existing secrets
- Fix secret-related security issues

## Core Security Principles

**CRITICAL RULES** - Never violated:

1. **No Secrets in .env**: Secrets MUST NEVER be in .env file
2. **No Secrets in docker-compose.yml**: No plaintext secrets in environment variables
3. **./secrets Directory**: All secrets in ./secrets with 700 permissions
4. **Secret Files**: Individual files with 600 permissions
5. **Git Protection**: ./secrets/* in .gitignore, never committed
6. **Proper Ownership**: All files owned by Docker user (not root)
7. **Docker Secrets Only**: Use Docker secrets mechanism exclusively

## Secret Management Workflow

### Phase 1: Understanding User Intent

**Step 1: Determine the Operation**

Ask yourself what the user wants to do:
- Create new secrets?
- Migrate existing environment variables to secrets?
- Validate current secret configuration?
- Audit secrets for leaks or issues?
- Update or rotate existing secrets?
- Remove secrets?

**Step 2: Gather Context**

1. Check current project state:
   - Does ./secrets directory exist?
   - Does docker-compose.yml exist?
   - Does .env file exist?
   - Is this part of stack-validator findings?

2. Review docker-compose.yml:
   - Any secrets already defined?
   - Any environment variables that look like secrets?
   - Which services need secrets?

3. Scan for security issues:
   - Secrets in .env?
   - Secrets in docker-compose.yml environment variables?
   - Secrets tracked in git?

### Phase 2: Secret Creation

**When**: User wants to create new secrets

**Step 1: Validate Prerequisites**

1. Check if ./secrets directory exists:
   ```bash
   ls -ld ./secrets
   ```
2. If missing, create with proper permissions:
   ```bash
   mkdir -p ./secrets
   chmod 700 ./secrets
   ```

**Step 2: Determine Secret Details**

Ask the user (or infer from context):
- Secret name (e.g., db_password, api_key)
- How to generate:
  - User provides value
  - Generate random value
  - Generate from pattern
- Format requirements (alphanumeric, hex, base64, etc.)
- Length requirements

**Step 3: Create Secret File**

1. Generate or accept secret value
2. Create file in ./secrets:
   ```bash
   echo -n "secret-value" > ./secrets/secret_name
   ```
3. Set proper permissions:
   ```bash
   chmod 600 ./secrets/secret_name
   ```
4. Verify ownership (should not be root)

**Step 4: Update docker-compose.yml**

1. Add to top-level secrets section:
   ```yaml
   secrets:
     secret_name:
       file: ./secrets/secret_name
   ```

2. Add to appropriate service:
   ```yaml
   services:
     myservice:
       secrets:
         - secret_name
   ```

**Step 5: Verify .gitignore**

Ensure ./secrets is excluded:
```gitignore
/secrets/
/secrets/*
!secrets/.gitkeep
```

### Phase 3: Secret Validation

**When**: User wants to validate secret configuration, or as part of other operations

**Step 1: Directory Structure Validation**

1. Check ./secrets exists:
   ```bash
   [ -d ./secrets ] && echo "exists" || echo "missing"
   ```

2. Check permissions (should be 700):
   ```bash
   stat -c "%a" ./secrets  # Linux
   stat -f "%OLp" ./secrets  # macOS
   ```

3. Check ownership (not root):
   ```bash
   ls -ld ./secrets
   ```

**Step 2: Secret Files Validation**

1. List all secret files:
   ```bash
   find ./secrets -type f ! -name .gitkeep
   ```

2. For each file, check:
   - Permissions (should be 600)
   - Ownership (not root)
   - Not empty
   - Readable

**Step 3: docker-compose.yml Validation**

1. Parse secrets section:
   - List all defined secrets
   - Verify files exist for each secret

2. Check service secret references:
   - All referenced secrets are defined
   - Services use `secrets:` key, not environment vars

3. **CRITICAL**: Scan for secrets in environment variables:
   - Look for patterns: PASSWORD, SECRET, KEY, TOKEN, API
   - Flag any that look like secrets
   - **These MUST be migrated**

**Step 4: .env File Validation**

1. **CRITICAL**: Scan .env for secrets:
   - Pattern matching: *PASSWORD*, *SECRET*, *KEY*, *TOKEN*, *API*
   - Long random-looking strings
   - Base64-encoded values
   - Any value that should be a secret

2. If secrets found in .env:
   - **This is a CRITICAL security issue**
   - List all detected secrets
   - Recommend immediate migration

**Step 5: Git Safety Check**

1. Verify .gitignore excludes ./secrets:
   ```bash
   grep -q "secrets" .gitignore
   ```

2. Check if any secrets are staged:
   ```bash
   git status --porcelain | grep secrets/
   ```

3. Check git history for secrets (if requested):
   ```bash
   git log --all --full-history -- ./secrets/
   ```

**Step 6: Generate Validation Report**

```
🔐 Secrets Validation Report
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

📁 Directory Structure
✅ ./secrets exists with 700 permissions
✅ Owned by user (1000:1000)
✅ ./secrets in .gitignore

📄 Secret Files (3)
✅ db_password - 600 permissions, 32 bytes
✅ api_key - 600 permissions, 64 bytes
⚠️  jwt_secret - 644 permissions (should be 600)

🐳 Docker Integration
✅ 3 secrets defined in docker-compose.yml
✅ All secret files exist
⚠️  Service 'worker' uses docker-entrypoint.sh

❌ CRITICAL SECURITY ISSUES
❌ .env contains secrets:
   * DB_PASSWORD=supersecret123
   * API_KEY=sk_live_abc123
   ** IMMEDIATE ACTION REQUIRED **

❌ docker-compose.yml environment variables contain secrets:
   * Service 'app' - JWT_SECRET in environment
   ** MUST MIGRATE TO DOCKER SECRETS **

✅ Git Safety
✅ No secrets in git staging
✅ .gitignore properly configured

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Status: FAILED (2 critical issues)

🔧 IMMEDIATE ACTIONS REQUIRED
1. Migrate secrets from .env to Docker secrets
2. Remove secrets from docker-compose.yml environment
3. Fix permissions on jwt_secret file
```

### Phase 4: Secret Migration

**When**: Secrets found in .env or docker-compose.yml environment variables

**CRITICAL**: This is a security issue that must be fixed

**Step 1: Identify Secrets to Migrate**

1. Scan .env for secret patterns:
   ```bash
   grep -E "(PASSWORD|SECRET|KEY|TOKEN|API)" .env
   ```

2. Scan docker-compose.yml environment sections:
   ```yaml
   # Look for patterns in environment variables
   ```

3. List all detected secrets with:
   - Variable name
   - Current location (.env or compose)
   - Current value (for migration)
   - Suggested secret name

**Step 2: Confirm with User**

Present findings and ask:
- Which variables should be migrated?
- Confirm secret names
- Confirm it's safe to remove from .env/compose

**Step 3: Create Secret Files**

For each secret to migrate:

1. Extract current value
2. Create secret file:
   ```bash
   echo -n "$value" > ./secrets/secret_name
   chmod 600 ./secrets/secret_name
   ```
3. Add to docker-compose.yml secrets section

**Step 4: Update Service Configurations**

For each service using the secret:

1. Add to service secrets list
2. Remove from environment variables
3. If container supports `_FILE` suffix:
   ```yaml
   environment:
     DB_PASSWORD_FILE: /run/secrets/db_password
   ```
4. If container doesn't support native secrets:
   - Create or update docker-entrypoint.sh
   - Document this requirement

**Step 5: Clean Up**

1. Remove secrets from .env:
   - Either delete the lines
   - Or comment them out with migration note
2. Remove from docker-compose.yml environment
3. Verify .env.example doesn't have secret values

**Step 6: Verification**

1. Test that services start correctly
2. Verify services can access secrets
3. Confirm no secrets remain in .env or compose
4. Run validation to confirm

### Phase 5: Secret Generation

**When**: Need to generate secure random secrets

**Step 1: Determine Format Requirements**

Common formats:
- **Alphanumeric**: Letters and numbers (default)
- **Hex**: Hexadecimal (0-9, a-f)
- **Base64**: Base64 encoding
- **Numeric**: Numbers only
- **UUID**: UUID v4 format

**Step 2: Determine Length**

Standard lengths:
- Database passwords: 32-64 characters
- API keys: 32-64 characters
- JWT secrets: 64 characters (or 32 bytes base64)
- Session secrets: 32 characters
- Encryption keys: 32 bytes (256-bit)

**Step 3: Generate Secret**

Use cryptographically secure methods:

```bash
# Alphanumeric (32 chars)
openssl rand -base64 32 | tr -d '/+=' | head -c 32

# Hex (64 chars)
openssl rand -hex 32

# Base64 (32 bytes)
openssl rand -base64 32

# UUID
uuidgen

# Custom (e.g., 16 alphanumeric)
LC_ALL=C tr -dc 'A-Za-z0-9' < /dev/urandom | head -c 16
```

**Step 4: Store Securely**

1. Write to file (no trailing newline):
   ```bash
   echo -n "$secret" > ./secrets/secret_name
   ```
2. Set permissions:
   ```bash
   chmod 600 ./secrets/secret_name
   ```

### Phase 6: Secret Auditing

**When**: User wants to audit secret usage, find leaks, or review security

**Step 1: Secret Inventory**

List all secrets with details:
- Name
- File path
- File size
- Permissions
- Owner
- Created date (if available)
- Last modified

**Step 2: Usage Analysis**

For each secret:
1. Check if defined in docker-compose.yml
2. List services using it
3. Check if file exists
4. Verify it's actually used

**Step 3: Find Unused Secrets**

1. Secrets defined but not used in any service
2. Secret files that aren't in docker-compose.yml
3. Suggest removal or documentation

**Step 4: Leak Detection**

Check common leak locations:

1. **.env file** (CRITICAL):
   ```bash
   grep -E "(PASSWORD|SECRET|KEY|TOKEN)" .env
   ```

2. **docker-compose.yml environment** (CRITICAL):
   - Scan all environment sections
   - Flag any secrets

3. **Configuration files**:
   ```bash
   grep -r "password\|secret\|key" ./config/
   ```

4. **Git history**:
   ```bash
   git log -p --all -S "secret-pattern"
   ```

5. **Docker logs**:
   - Check recent logs for secret exposure

**Step 5: Permission Audit**

1. Check all files in ./secrets:
   ```bash
   find ./secrets -type f -not -perm 600
   ```

2. Check directory permissions:
   ```bash
   [ "$(stat -c '%a' ./secrets)" = "700" ]
   ```

3. Check ownership:
   ```bash
   find ./secrets -user root
   ```

**Step 6: Generate Audit Report**

Include:
- Total secrets count
- Usage statistics
- Security issues found
- Recommendations
- Risk assessment

### Phase 7: docker-entrypoint.sh Generation

**When**: Container doesn't support native Docker secrets

**Step 1: Determine Necessity**

Check if container supports secrets:
- PostgreSQL, MySQL, MariaDB: Support `_FILE` suffix ✅
- Redis: Native secret support ✅
- MongoDB: Native secret support ✅
- Most modern containers: Check documentation

Only create entrypoint if:
- Container expects environment variables only
- No `_FILE` suffix support
- No native /run/secrets/ reading

**Step 2: Identify Required Secrets**

List secrets that need to be loaded:
- Secret name (in ./secrets/)
- Environment variable name
- Service name

**Step 3: Generate Entrypoint Script**

```bash
#!/bin/bash
set -e

# Function to load secrets from docker secrets into environment
load_secret() {
  local secret_name=$1
  local env_var=$2
  local secret_file="/run/secrets/${secret_name}"

  if [ -f "$secret_file" ]; then
    export "${env_var}=$(cat "$secret_file")"
    echo "Loaded secret: $secret_name -> $env_var"
  else
    echo "ERROR: Secret file $secret_file not found!" >&2
    exit 1
  fi
}

# Load all required secrets
load_secret "db_password" "DB_PASSWORD"
load_secret "api_key" "API_KEY"
load_secret "jwt_secret" "JWT_SECRET"

# Execute the main command
exec "$@"
```

**Step 4: Set Permissions**

```bash
chmod +x docker-entrypoint.sh
```

**Step 5: Update docker-compose.yml**

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

**Step 6: Document**

Add comment explaining why entrypoint is needed:
```yaml
# docker-entrypoint.sh required because this container
# doesn't support Docker secrets natively
```

## Communication Style

When managing secrets:

1. **Be Security-Focused**: Emphasize security at every step
2. **Be Clear About Risks**: Explain why secrets in .env/compose is dangerous
3. **Be Urgent About Critical Issues**: Don't downplay security problems
4. **Be Helpful**: Provide exact commands to fix issues
5. **Be Thorough**: Check all potential leak locations
6. **Be Educational**: Explain why Docker secrets are better
7. **Never Display Secret Values**: Show "[REDACTED]" instead

## Critical Validation Points

These are **must-pass** security criteria:

1. ✅ NO secrets in .env file
2. ✅ NO secrets in docker-compose.yml environment variables
3. ✅ ./secrets directory exists with 700 permissions
4. ✅ All secret files have 600 permissions
5. ✅ ./secrets/* in .gitignore
6. ✅ No secrets tracked in git
7. ✅ No root-owned secret files
8. ✅ All referenced secrets exist
9. ✅ docker-entrypoint.sh only when truly necessary

## Integration with Companion Skills

### stack-validator
- Stack-validator calls this skill for secret validation
- Validates that secrets follow proper patterns
- Detects secrets in .env and compose files

### stack-creator
- Creates ./secrets directory with proper setup
- Generates .gitkeep file
- Sets up .gitignore correctly
- Creates initial secret placeholders

### config-generator
- Ensures configs don't contain secrets
- References secrets properly
- Uses environment variables for non-secrets only

## Important Notes

- **Read-Only for Secrets**: NEVER display actual secret values to user
- **Security First**: Always prioritize security over convenience
- **Migration Required**: Secrets in .env/compose MUST be migrated
- **No Shortcuts**: Always follow security best practices
- **Verify Everything**: Check permissions, ownership, git status
- **Companion-Aware**: Work with other skills seamlessly

## Example Workflow: Complete Secret Setup

```
User: "Set up secrets for my database"

1. Check current state
   - ./secrets missing → create it
   - docker-compose.yml has DB_PASSWORD in environment → CRITICAL ISSUE

2. Report findings:
   "I found a critical security issue: DB_PASSWORD is in docker-compose.yml
    environment variables. I'll migrate this to Docker secrets."

3. Migration:
   - Extract password value
   - Create ./secrets/db_password
   - chmod 600 ./secrets/db_password
   - Update docker-compose.yml secrets section
   - Update postgres service to use secrets
   - Remove from environment

4. Verification:
   - Run validation
   - Confirm no secrets in compose
   - Check .gitignore
   - Verify permissions

5. Report:
   "✅ Database password now secured with Docker secrets
    ✅ Removed from docker-compose.yml environment
    ✅ File permissions set correctly
    ✅ Added to .gitignore"
```

---

*This skill ensures secrets are managed securely and never exposed in configuration files or version control.*
