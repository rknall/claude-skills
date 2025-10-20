# GitLab Stack Validator

> Comprehensive validation for GitLab stack projects to ensure deployment readiness and best practices compliance

## Overview

The GitLab Stack Validator skill provides thorough validation of stack projects before deployment. It checks directory structure, environment variable configuration, secrets management, Docker configuration, and file ownership to ensure your stack follows proper architecture patterns and is ready for production.

**Key Focus**: Detection and reporting of issues (not automatic fixes)

## Features

- **Comprehensive Validation**: 8 validation categories covering all aspects of stack configuration
- **Environment Variable Synchronization**: Critical .env and .env.example sync checking
- **Secrets Security**: Ensures secrets are properly isolated and secured
- **Docker Best Practices**: Leverages docker-validation skill for Docker-specific checks
- **File Ownership Auditing**: Detects problematic root-owned files
- **Detailed Reporting**: Clear, actionable findings with severity levels
- **Integration Ready**: Works alongside stack-creator and secrets-manager skills

## When to Use

Use this skill when you need to:

- Validate a GitLab stack project before deployment
- Verify stack configuration and architecture
- Audit stack for security and best practices compliance
- Pre-deployment health checks
- Ensure stack follows proper patterns
- Identify configuration issues before they cause problems

## Installation

```bash
# Install the skill marketplace
/plugin marketplace add rknall/Skills

# Install the stack-validator skill
/plugin install stack-validator
```

## Quick Start

### Basic Validation

```bash
# Validate current directory
claude "validate this stack"

# Validate specific directory
claude "validate the stack in /path/to/my-stack"
```

### Full Validation Example

```bash
# In your stack directory
cd ~/projects/my-stack

# Run validation
claude "validate this stack"
```

**Example Output:**

```
🔍 GitLab Stack Validation Report
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Stack: my-stack
Date: 2025-01-15 14:30:22
Mode: standard

📊 SUMMARY
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
✅ Passed: 5
⚠️  Warnings: 2
❌ Critical: 1
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

📋 DETAILED FINDINGS
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

✅ Directory Structure: PASS

❌ Environment Variables: CRITICAL
   ❌ .env.example file not found
      Impact: No template for environment setup
      Details: Required for documenting configuration

⚠️  File Ownership: WARNING
   ⚠️  Root-owned files detected
      Location: ./config/nginx.conf
      Impact: Cannot modify without sudo
      Details: Owner: root, Expected: user

✅ Docker Configuration: PASS
✅ Secrets Management: PASS
✅ Configuration Files: PASS
✅ Scripts: PASS
✅ Temporary Directory: PASS

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

🎯 OVERALL STATUS: FAILED

🔧 RECOMMENDED ACTIONS
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
1. Create .env.example file (use stack-creator skill)
2. Fix ownership of ./config/nginx.conf:
   sudo chown $(id -u):$(id -g) ./config/nginx.conf

💡 NEXT STEPS
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
- Use stack-creator skill to generate .env.example
- Re-run validation after fixes
```

## What Gets Validated

### 1. Directory Structure
- ✅ Required directories exist (./config, ./secrets, ./_temporary)
- ✅ Proper permissions on ./secrets (700)
- ✅ .gitignore excludes secrets and temporary files
- ✅ No root-owned directories

### 2. Environment Variables
- ✅ .env file exists and is valid
- ✅ .env.example exists and is valid
- ✅ **CRITICAL**: .env and .env.example are synchronized
- ✅ No secrets detected in .env
- ✅ .env is not tracked by git

### 3. Docker Configuration
- ✅ docker-compose.yml valid (via docker-validation skill)
- ✅ No deprecated version field
- ✅ Secrets properly defined in top-level section
- ✅ Services reference secrets correctly
- ✅ Volume mounts follow patterns
- ✅ Service dependencies declared

### 4. Secrets Management
- ✅ ./secrets directory exists and secure (700 permissions)
- ✅ All referenced secret files exist
- ✅ Secret files have proper permissions (600)
- ✅ No secrets in environment variables
- ✅ No hardcoded secrets anywhere

### 5. Configuration Files
- ✅ Proper organization in ./config
- ✅ Valid syntax (YAML, JSON, INI, TOML)
- ✅ No embedded secrets
- ✅ Correct file ownership

### 6. Scripts
- ✅ docker-entrypoint.sh only when necessary
- ✅ Executable permissions set
- ✅ No hardcoded secrets
- ✅ Proper error handling

### 7. Temporary Directory
- ✅ ./_temporary exists
- ✅ Excluded from git
- ✅ Empty or contains only expected files
- ✅ Properly utilized in compose file

### 8. File Ownership
- ✅ No root-owned files
- ✅ Consistent ownership (Docker user)
- ✅ Proper permissions throughout

## Validation Modes

### Standard Mode (Default)
Reports all issues as warnings or errors with full details.

```bash
claude "validate this stack"
```

### Strict Mode
Fails on any warnings - zero tolerance for deviations.

```bash
claude "validate this stack in strict mode"
```

### Targeted Validation
Validate specific categories only.

```bash
# Validate only secrets
claude "validate secrets configuration"

# Validate only environment variables
claude "check .env configuration"

# Validate only Docker setup
claude "validate docker configuration"
```

## Critical Validation Points

These are **must-pass** criteria for production deployment:

1. ✅ .env and .env.example are fully synchronized
2. ✅ No secrets in docker-compose.yml environment variables
3. ✅ ./secrets directory exists with restrictive permissions (700)
4. ✅ All referenced secrets exist in ./secrets
5. ✅ ./secrets and ./_temporary are in .gitignore
6. ✅ No root-owned files in project
7. ✅ docker-compose.yml passes docker-validation checks
8. ✅ No secrets exposed in git
9. ✅ .env file is not tracked by git

## Integration with Companion Skills

### stack-creator
**When to Use**: Fix structural issues, create missing files

```bash
# After validation finds missing .env.example
claude "use stack-creator to add .env.example"

# Fix directory structure
claude "use stack-creator to set up required directories"
```

### secrets-manager
**When to Use**: Configure secrets, fix secret-related issues

```bash
# After validation finds secret issues
claude "use secrets-manager to properly configure db_password"

# Move secrets from .env to Docker secrets
claude "use secrets-manager to migrate secrets from .env"
```

### docker-validation
**Automatic Integration**: Automatically invoked during stack validation

The stack-validator automatically uses the docker-validation skill to check:
- docker-compose.yml syntax and best practices
- Dockerfile validation (if present)
- Multi-stage build verification
- Security configurations

## Common Issues and Fixes

### Issue: .env.example Missing

**Validation Output:**
```
❌ Environment Variables: CRITICAL
   - .env.example file not found
```

**Fix:**
```bash
claude "use stack-creator to generate .env.example from my .env"
```

---

### Issue: .env and .env.example Out of Sync

**Validation Output:**
```
❌ Environment Variables: CRITICAL
   - Variables in .env but NOT in .env.example:
     * API_TIMEOUT
   - Variables in .env.example but NOT in .env:
     * CACHE_TTL
```

**Fix:**
Manually add missing variables to both files, or:
```bash
claude "use stack-creator to synchronize .env.example with .env"
```

---

### Issue: Secrets in .env File

**Validation Output:**
```
❌ Environment Variables: CRITICAL
   - Potential secrets detected in .env:
     * DB_PASSWORD
     * API_KEY
```

**Fix:**
```bash
claude "use secrets-manager to move DB_PASSWORD and API_KEY to Docker secrets"
```

---

### Issue: Root-Owned Files

**Validation Output:**
```
❌ File Ownership: CRITICAL
   - Root-owned files:
     * ./config/nginx.conf
```

**Fix:**
```bash
sudo chown $(id -u):$(id -g) ./config/nginx.conf
```

---

### Issue: Missing ./secrets Directory

**Validation Output:**
```
❌ Directory Structure: CRITICAL
   - Required directory './secrets' not found
```

**Fix:**
```bash
claude "use stack-creator to create required directories"
```

---

## Custom Validation Rules

Create `.stack-validator.yml` in your project root to customize validation:

```yaml
# Validation mode
strict_mode: false

# Whether warnings should fail validation
fail_on_warnings: false

# Paths to exclude from validation
exclude_paths:
  - ./vendor
  - ./node_modules
  - ./_temporary/cache

# Specific checks to skip
skip_checks:
  - temporary-directory-empty
```

## Output Formats

### Text Report (Default)
Human-readable validation report with colored output and formatting.

```bash
claude "validate this stack"
```

### JSON Output
Machine-readable format for CI/CD integration.

```bash
claude "validate this stack and output JSON"
```

**JSON Structure:**
```json
{
  "stack": "my-stack",
  "timestamp": "2025-01-15T14:30:22Z",
  "summary": {
    "passed": 5,
    "warnings": 2,
    "critical": 1,
    "status": "failed"
  },
  "findings": [
    {
      "category": "environment-variables",
      "status": "critical",
      "issues": [
        {
          "severity": "critical",
          "message": ".env.example file not found",
          "impact": "No template for environment setup"
        }
      ]
    }
  ]
}
```

## CI/CD Integration

### GitLab CI Example

```yaml
validate-stack:
  stage: validate
  image: anthropic/claude-code:latest
  script:
    - cd $CI_PROJECT_DIR
    - claude "validate this stack in strict mode"
  only:
    - merge_requests
    - main
```

### GitHub Actions Example

```yaml
- name: Validate Stack
  run: |
    cd $GITHUB_WORKSPACE
    claude "validate this stack"
```

## Architecture Patterns

This skill validates stacks following these principles:

1. **Configuration Management**: All configuration in docker-compose.yml and ./config
2. **Secrets Isolation**: Secrets in ./secrets via Docker secrets
3. **Environment Variables**: .env with matching .env.example
4. **Minimal Scripts**: docker-entrypoint.sh only when necessary
5. **Proper Ownership**: No root-owned files
6. **Temporary Files**: ./_temporary for transient data
7. **Docker Best Practices**: Modern compose syntax, proper volume mounts

See [validation-patterns.md](validation-patterns.md) for detailed patterns and examples.

## Troubleshooting

### Validation Says "Not a Stack Project"

**Cause**: Missing docker-compose.yml or stack indicators

**Fix**:
- Ensure you're in the correct directory
- Check that docker-compose.yml exists
- Initialize a new stack with stack-creator if needed

---

### Validation Finds Many Root-Owned Files

**Cause**: Containers running as root and creating files in mounted volumes

**Fix**:
1. Fix existing files: `sudo chown -R $(id -u):$(id -g) .`
2. Prevent future issues: Run containers as current user
   ```yaml
   services:
     app:
       user: "${UID}:${GID}"
   ```

---

### Docker Validation Fails

**Cause**: docker-compose.yml has syntax or best practice issues

**Fix**: The docker-validation skill will provide specific guidance. Follow its recommendations.

---

## Reference Documentation

- [SKILL.md](SKILL.md) - Complete skill workflow and implementation
- [validation-patterns.md](validation-patterns.md) - Architecture patterns and examples
- [common-issues.md](common-issues.md) - Detailed issue explanations and fixes

## Version History

### v1.0.0 (2025-01-15)
- Initial release
- 8 comprehensive validation categories
- .env and .env.example synchronization checking
- Integration with docker-validation skill
- Support for companion skills (stack-creator, secrets-manager)
- Custom validation rules support
- JSON output format

## Contributing

Found an issue or have a suggestion? This skill is part of the rknall-custom-skills marketplace.

## License

Part of the rknall-custom-skills marketplace for Claude Code.

---

**Ensure your GitLab stacks are secure, compliant, and deployment-ready with comprehensive validation.**
