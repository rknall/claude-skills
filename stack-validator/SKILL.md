---
name: "GitLab Stack Validator"
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

**Step 1: Verify Stack Project**
1. Check if current directory appears to be a stack project
2. Look for key indicators:
   - docker-compose.yml exists
   - Presence of ./config, ./secrets, or ./_temporary directories
   - .env file exists
3. If not a stack project, report findings and ask if user wants to initialize one

**Step 2: Gather Context**
1. Ask user for validation scope (if needed):
   - Full validation or targeted checks?
   - Strict mode or permissive mode?
   - Output format preference (text report vs JSON)?
2. Check if .stack-validator.yml exists for custom rules
3. Note the working directory for reporting

### Phase 2: Directory Structure Validation

**Step 1: Required Directories**
1. Check for required directories:
   - `./config` - Configuration files directory
   - `./secrets` - Secrets storage directory
   - `./_temporary` - Temporary files directory
2. For each missing directory, record:
   - Directory name
   - Purpose and why it's required
   - Impact on stack functionality

**Step 2: Directory Permissions**
1. Check ./secrets has restricted permissions (700 or 600)
2. Verify other directories have appropriate permissions
3. Report any permission issues with security implications

**Step 3: Directory Ownership**
1. Scan all project directories for ownership
2. Flag any root-owned directories:
   - Directory path
   - Current owner
   - Expected owner (current user or docker user)
3. Flag any files with unexpected ownership

**Step 4: .gitignore Validation**
1. Check if .gitignore exists
2. Verify it excludes:
   - `./secrets` or `./secrets/*`
   - `./_temporary` or `./_temporary/*`
   - `.env` (should not be in git)
3. Report missing exclusions with security implications

### Phase 3: Environment Variables Validation

**Step 1: .env File Validation**
1. Check if .env file exists
2. Parse .env file for:
   - Syntax errors
   - Duplicate variable definitions
   - Empty values that might be required
3. Record all environment variable names found

**Step 2: .env.example Validation**
1. Check if .env.example exists
2. If missing, this is a **critical issue** - document why it's needed
3. Parse .env.example for variable names

**Step 3: .env Synchronization Check**
1. Compare variables in .env with .env.example:
   - Variables in .env but NOT in .env.example
   - Variables in .env.example but NOT in .env
2. This is a **critical validation point** - they must match
3. Report any mismatches with specific variable names

**Step 4: Environment Variable Security**
1. Scan .env for potential secrets:
   - Variables with names containing: password, secret, key, token, api
   - Base64-encoded looking values
   - Long random strings
2. If secrets detected in .env, flag as security issue
3. Suggest moving to ./secrets and Docker secrets instead

### Phase 4: Docker Configuration Validation

**Step 1: Invoke docker-validation Skill**
1. Use the docker-validation skill to validate:
   - docker-compose.yml syntax and best practices
   - Dockerfile(s) if present
   - Multi-stage builds
   - Security configurations
2. Collect all findings from docker-validation
3. Integrate into overall stack validation report

**Step 2: Stack-Specific Docker Checks**
1. Review docker-compose.yml for stack patterns:
   - Secrets are defined in top-level `secrets:` section
   - Services reference secrets via `secrets:` key (not environment variables)
   - Volume mounts follow patterns (./config, ./secrets, ./_temporary)
   - Networks are properly defined if multi-service
   - Service dependencies use `depends_on` correctly

**Step 3: Version Check**
1. Ensure docker-compose.yml does NOT have version field at top
2. This follows modern Docker Compose specification
3. If version field present, flag for removal

### Phase 5: Secrets Management Validation

**Step 1: Secrets Directory Check**
1. Verify ./secrets directory exists
2. Check permissions are restrictive (700)
3. Verify it's excluded from git

**Step 2: Docker Secrets Validation**
1. Parse docker-compose.yml for secrets definitions
2. For each secret definition:
   - Verify the secret file exists in ./secrets
   - Check file permissions (600 or 400)
   - Verify not tracked by git
3. Report missing secret files

**Step 3: Secret References Validation**
1. Check each service's `secrets:` section
2. Verify referenced secrets are defined in top-level secrets
3. Flag any undefined secret references

**Step 4: Environment Variables vs Secrets**
1. Scan service environment variables for potential secrets
2. Look for patterns like:
   - *_PASSWORD
   - *_SECRET
   - *_KEY
   - *_TOKEN
   - API_*
3. If sensitive data in environment, suggest using secrets instead

**Step 5: Secret Exposure Check**
1. Check for secrets in:
   - docker-compose.yml (hardcoded values)
   - .env file (should be in ./secrets instead)
   - Configuration files in ./config
   - Any shell scripts
2. Report any exposed secrets as **critical security issues**

### Phase 6: Configuration Files Validation

**Step 1: Config Directory Structure**
1. List all files in ./config directory
2. Organize by service (if applicable)
3. Check for proper organization

**Step 2: Config File Validation**
1. For common config file types, validate syntax:
   - YAML files (.yml, .yaml)
   - JSON files (.json)
   - INI files (.ini, .conf)
   - TOML files (.toml)
2. Report any syntax errors

**Step 3: Config vs Secrets Separation**
1. Scan config files for potential secrets
2. Look for hardcoded passwords, tokens, keys
3. Flag any secrets that should be in ./secrets instead

**Step 4: Config Ownership**
1. Check all config files for ownership
2. Flag any root-owned config files
3. Report expected ownership (current user)

### Phase 7: Script Validation

**Step 1: docker-entrypoint.sh Detection**
1. Search for docker-entrypoint.sh files
2. For each found:
   - Document location
   - Check if truly necessary
3. Validate it's only used when container doesn't support native secrets

**Step 2: Script Permissions**
1. Check docker-entrypoint.sh is executable (chmod +x)
2. Verify ownership (should not be root)
3. Report permission issues

**Step 3: Script Content Validation**
1. Scan script for:
   - Hardcoded secrets (critical issue)
   - Proper secret handling from /run/secrets/
   - Error handling
   - Syntax errors (if bash/sh)
2. Report any issues found

**Step 4: Necessity Check**
1. Review if docker-entrypoint.sh is truly needed
2. Check if service supports native Docker secrets
3. Suggest removal if unnecessary

### Phase 8: Temporary Directory Validation

**Step 1: Directory Check**
1. Verify ./_temporary exists
2. Check it's in .gitignore
3. Verify permissions

**Step 2: Content Check**
1. List contents of ./_temporary
2. Check for:
   - Leftover files that should be cleaned
   - Large files consuming space
   - Old files (> 7 days old)
3. Report if not empty with details

**Step 3: Usage Validation**
1. Check if ./_temporary is properly mounted in docker-compose.yml
2. Verify services use it for temporary files
3. Flag if not being utilized

### Phase 9: File Ownership Audit

**Step 1: Comprehensive Ownership Scan**
1. Use `find . -type f -user root 2>/dev/null` to find root-owned files
2. Exclude expected directories (.git, node_modules, vendor)
3. List all root-owned files found

**Step 2: Categorize Issues**
1. Group by directory:
   - ./config files owned by root
   - ./secrets files owned by root
   - Application files owned by root
2. Report with specific paths

**Step 3: Impact Assessment**
1. For each root-owned file:
   - Explain why this is problematic
   - Impact on stack operations
   - Potential errors that may occur

### Phase 10: Report Generation

**Step 1: Compile All Findings**
1. Organize findings by category:
   - Directory Structure
   - Environment Variables (.env sync)
   - Docker Configuration
   - Secrets Management
   - Configuration Files
   - Scripts (docker-entrypoint.sh)
   - Temporary Directory
   - File Ownership
2. Assign severity levels:
   - ❌ **CRITICAL**: Security issues, missing required components, .env mismatch
   - ⚠️ **WARNING**: Best practice violations, potential issues
   - ✅ **PASS**: No issues found

**Step 2: Generate Validation Report**

Create a comprehensive report with this structure:

```
🔍 GitLab Stack Validation Report
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Stack: [directory name]
Date: [timestamp]
Mode: [strict/permissive]

📊 SUMMARY
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
✅ Passed: [count]
⚠️  Warnings: [count]
❌ Critical: [count]
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

📋 DETAILED FINDINGS
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

[For each validation category]

[Category Icon] [Category Name]: [STATUS]
[If issues found:]
   ❌ [Issue description]
      Location: [file/directory path]
      Impact: [what this affects]
      Details: [specific information]

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

🎯 OVERALL STATUS: [PASSED/FAILED/WARNINGS]

[If failures or warnings:]
🔧 RECOMMENDED ACTIONS
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
[Prioritized list of actions needed]
1. [Most critical action]
2. [Next action]
...

💡 NEXT STEPS
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
- Use stack-creator skill to fix structural issues
- Use secrets-manager skill to properly configure secrets
- Re-run validation after fixes
```

**Step 3: Provide Actionable Guidance**
1. For each issue, provide:
   - Clear description of the problem
   - Why it matters
   - Which companion skill can help fix it
   - General guidance (but not actual fix commands)
2. Prioritize issues by severity
3. Group related issues together

**Step 4: Export Options**
1. If user requested JSON output, export as:
   ```json
   {
     "stack": "directory-name",
     "timestamp": "ISO-8601",
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
         "issues": [...]
       }
     ]
   }
   ```

## Validation Categories Reference

### 1. Directory Structure
- Required directories exist
- Proper permissions
- .gitignore coverage
- Ownership correctness

### 2. Environment Variables
- .env file exists and valid
- .env.example exists
- **CRITICAL**: .env and .env.example are synchronized
- No secrets in .env

### 3. Docker Configuration
- docker-compose.yml valid (via docker-validation skill)
- No version field
- Secrets properly defined
- Volumes follow patterns
- Service dependencies correct

### 4. Secrets Management
- ./secrets directory secure
- All referenced secrets exist
- Proper permissions
- No exposed secrets
- Environment variables don't contain secrets

### 5. Configuration Files
- Proper organization
- Valid syntax
- No embedded secrets
- Correct ownership

### 6. Scripts
- docker-entrypoint.sh only when necessary
- Executable permissions
- No hardcoded secrets
- Proper secret handling

### 7. Temporary Directory
- Exists and in .gitignore
- Empty or contains only expected transient files
- Properly utilized in compose file

### 8. File Ownership
- No root-owned files
- Consistent ownership
- Proper user/group

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

### Standard Mode (Default)
- Report all issues as warnings or errors
- Provide comprehensive findings
- Suggest improvements

### Strict Mode
- Fail on warnings
- Require all best practices
- Zero tolerance for deviations

### Permissive Mode
- Only fail on critical issues
- Allow warnings
- More lenient on best practices

## Configuration Support

If `.stack-validator.yml` exists in project root, respect these settings:

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

# Custom validation scripts to run
custom_checks:
  - ./scripts/custom-validation.sh

# Specific checks to skip
skip_checks:
  - temporary-directory-empty
```

## Communication Style

When reporting validation results:

1. **Be Clear and Direct**: State issues plainly without hedging
2. **Be Specific**: Include exact file paths, line numbers, variable names
3. **Be Actionable**: Explain what needs to be done (without doing it)
4. **Be Helpful**: Point to companion skills that can fix issues
5. **Be Organized**: Group related issues, prioritize by severity
6. **Be Educational**: Explain why something is an issue
7. **Be Concise**: Don't overwhelm with details, but be thorough

## Critical Validation Points

These are **must-pass** criteria for production stacks:

1. ✅ .env and .env.example are fully synchronized
2. ✅ No secrets in docker-compose.yml environment variables
3. ✅ ./secrets directory exists and has restrictive permissions
4. ✅ All referenced secrets exist
5. ✅ ./secrets and ./_temporary are in .gitignore
6. ✅ No root-owned files
7. ✅ docker-compose.yml passes docker-validation
8. ✅ No secrets exposed in git
9. ✅ .env file is not tracked by git

## Important Notes

- **Read-Only**: This skill NEVER modifies files - validation only
- **Detection-Focused**: Report issues, don't fix them
- **Comprehensive**: Check all aspects systematically
- **Fast**: Complete validation in seconds
- **Integrative**: Use docker-validation skill for Docker checks
- **Companion-Aware**: Direct users to appropriate skills for fixes

## Example Validation Flow

```
User: "Validate my stack"

1. Check if docker-compose.yml exists ✅
2. Verify directory structure
   - ./config ✅
   - ./secrets ✅
   - ./_temporary ⚠️ (not in .gitignore)
3. Validate .env configuration
   - .env exists ✅
   - .env.example exists ❌ CRITICAL
4. Run docker-validation skill
   - docker-compose.yml syntax ✅
   - Secrets configuration ⚠️ (using environment vars)
5. Check secrets management
   - ./secrets permissions ✅
   - Secret files exist ✅
6. Scan for ownership issues
   - Found 3 root-owned files in ./config ❌
7. Generate report with findings
8. Suggest: Use stack-creator to add .env.example and fix .gitignore
```

---

*This skill ensures stack quality, security, and deployment readiness through comprehensive validation.*
