---
name: "stack-validator"
description: "Validates GitLab stack projects before deployment, ensuring proper architecture patterns, directory structure, secrets management, .env configuration, and Docker best practices. Use when users ask to validate a stack, check stack configuration, verify stack architecture, audit stack setup, or ensure stack deployment readiness."
---

# GitLab Stack Validator

This skill validates GitLab stack projects to ensure they follow proper architecture patterns and are ready for deployment. It focuses on **detection and reporting** of issues, working alongside companion skills (stack-creator, secrets-manager) for remediation.

## When to Use This Skill

Activate this skill when the user requests:
- Validate a GitLab stack project
- Check stack configuration before deployment
- Verify stack architecture and structure
- Audit stack for best practices compliance
- Ensure stack follows proper patterns
- Pre-deployment validation checks
- Stack health check or readiness verification

## Core Validation Principles

This skill validates stacks that follow these architecture principles:

1. **Configuration Management**: All configuration through docker-compose.yml and ./config directory
2. **Secrets Management**: Secrets stored in ./secrets and referenced via Docker secrets
3. **Environment Variables**: .env file with matching .env.example template
4. **Minimal Custom Scripts**: docker-entrypoint.sh only when containers don't support native secrets
5. **Proper Ownership**: No root-owned files (all files owned by Docker user)
6. **Temporary Files**: _temporary directory for transient files that are cleaned up after use
7. **Docker Best Practices**: Leverage docker-validation skill for Docker-specific checks

## Validation Workflow

When a user requests stack validation, follow this comprehensive workflow:

### Phase 1: Pre-flight Checks

1. Verify stack project exists:
   ```bash
   ls docker-compose.yml .env ./config ./secrets ./_temporary 2>/dev/null
   ```
2. If not a stack project, report findings and ask if user wants to initialize one
3. Check for `.stack-validator.yml` custom rules
4. Confirm validation scope with user if needed (full/targeted, strict/permissive, text/JSON output)

### Phase 2: Directory Structure Validation

1. Check required directories exist: `./config`, `./secrets`, `./_temporary`
2. Verify permissions:
   ```bash
   stat -c '%a %U:%G %n' ./config ./secrets ./_temporary 2>/dev/null
   # ./secrets must be 700; flag any root-owned directories
   ```
3. Validate `.gitignore` includes: `secrets/`, `_temporary/`, `.env`
   ```bash
   grep -E '(secrets|_temporary|\.env)' .gitignore
   ```
4. Record missing directories, permission violations, and ownership issues with paths

### Phase 3: Environment Variables Validation

1. Parse `.env` for syntax errors, duplicates, and empty required values
2. **CRITICAL**: Verify `.env.example` exists — missing template is a critical issue
3. **CRITICAL**: Check `.env` and `.env.example` are synchronized:
   ```bash
   diff <(grep -oP '^[A-Z_]+' .env | sort) <(grep -oP '^[A-Z_]+' .env.example | sort)
   ```
   Report specific variable names that are missing from either side.
4. Scan `.env` for secrets (variables matching `*_PASSWORD`, `*_SECRET`, `*_KEY`, `*_TOKEN`, `API_*`). Flag and suggest moving to `./secrets` with Docker secrets.

### Phase 4: Docker Configuration Validation

1. **Invoke docker-validation skill** for syntax, Dockerfile, multi-stage build, and security checks. Integrate findings into the stack report.
2. Stack-specific docker-compose.yml checks:
   - Top-level `secrets:` section defines all secrets
   - Services reference secrets via `secrets:` key (not environment variables)
   - Volume mounts use `./config`, `./secrets`, `./_temporary` patterns
   - Networks defined for multi-service stacks; `depends_on` used correctly
3. Flag if `version:` field is present (removed in modern Compose spec):
   ```bash
   grep -n '^version:' docker-compose.yml
   ```

### Phase 5: Secrets Management Validation

1. Verify `./secrets` exists with restrictive permissions and is git-ignored:
   ```bash
   stat -c '%a' ./secrets && git check-ignore ./secrets
   ```
2. For each secret defined in docker-compose.yml, verify the file exists and has proper permissions:
   ```bash
   find ./secrets -type f -exec stat -c '%a %n' {} \;
   # Files should be 600 or 400; flag any tracked by git
   ```
3. Verify all service-level `secrets:` references resolve to top-level secret definitions
4. Scan for secret exposure: hardcoded values in docker-compose.yml, `.env`, `./config/*`, and shell scripts — flag as **critical security issues**

### Phase 6: Configuration Files Validation

1. List and validate syntax of config files by type:
   ```bash
   find ./config -type f \( -name '*.yml' -o -name '*.yaml' -o -name '*.json' -o -name '*.toml' -o -name '*.ini' \)
   ```
   Use `python3 -c "import yaml; yaml.safe_load(open('file'))"` or `jq . file.json` for syntax checks.
2. Scan config files for hardcoded secrets (passwords, tokens, keys) — flag for migration to `./secrets`
3. Check ownership — flag any root-owned config files

### Phase 7: Script Validation

1. Find entrypoint scripts and check permissions:
   ```bash
   find . -name 'docker-entrypoint.sh' -exec stat -c '%a %U %n' {} \;
   # Must be executable (+x), not root-owned
   ```
2. Validate script content: check for hardcoded secrets, proper `/run/secrets/` handling, error handling, and shell syntax (`bash -n script.sh`)
3. Assess necessity — if the container supports native Docker secrets, recommend removing the entrypoint script

### Phase 8: Temporary Directory Validation

1. Verify `./_temporary` exists, is in `.gitignore`, and has correct permissions
2. Check for stale content:
   ```bash
   find ./_temporary -type f -mtime +7 -ls  # Files older than 7 days
   du -sh ./_temporary                       # Total size
   ```
3. Verify `./_temporary` is mounted in docker-compose.yml — flag if unused

### Phase 9: File Ownership Audit

1. Scan for root-owned files:
   ```bash
   find . -type f -user root -not -path './.git/*' -not -path './node_modules/*' -not -path './vendor/*' 2>/dev/null
   ```
2. Group findings by directory (`./config`, `./secrets`, application files) and report specific paths
3. For each root-owned file, note the operational impact (container permission errors, failed mounts)

### Phase 10: Report Generation

**Severity levels:**
- ❌ **CRITICAL**: Security issues, missing required components, .env mismatch
- ⚠️ **WARNING**: Best practice violations, potential issues
- ✅ **PASS**: No issues found

**Report structure:**
1. **Summary**: Stack name, date, mode, counts of passed/warning/critical checks
2. **Detailed findings**: Grouped by category (Phases 2-9), each with status, issue description, file path, and impact
3. **Recommended actions**: Prioritized list with companion skill references (stack-creator for structural fixes, secrets-manager for secret issues)
4. **JSON export** (if requested): `{ "stack", "timestamp", "summary": { "passed", "warnings", "critical", "status" }, "findings": [...] }`

For each issue, include the specific file path, what's wrong, and which companion skill can remediate it.

## Integration with Companion Skills

### stack-creator
- Recommend for: Creating missing directories, initializing structure
- Handles: Setting up new stacks, fixing structural issues

### secrets-manager
- Recommend for: Secrets configuration, secret file management
- Handles: Creating/updating secrets, proper secret setup

### docker-validation
- **Used directly during validation** for Docker-specific checks
- Provides: Docker Compose and Dockerfile validation
- Integrated into stack validation report

## Validation Modes

| Mode | Behaviour |
|------|-----------|
| **Standard** (default) | Report all issues; suggest improvements |
| **Strict** | Fail on warnings; require all best practices |
| **Permissive** | Fail only on critical issues; allow warnings |

If `.stack-validator.yml` exists, respect its `strict_mode`, `fail_on_warnings`, `exclude_paths`, `custom_checks`, and `skip_checks` settings.

## Reference Materials

- **[Validation Patterns](./validation-patterns.md)**: Detailed architecture patterns, expected directory layouts, docker-compose patterns, and secrets management patterns
- **[Common Issues](./common-issues.md)**: Catalogue of frequent validation failures with causes, impacts, and remediation guidance

**Behaviour**: This skill is read-only — it detects and reports issues but never modifies files. Direct users to companion skills for remediation.
