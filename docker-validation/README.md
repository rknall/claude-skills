# Docker Configuration Validator

**Version:** 1.0.0
**Category:** Development
**Description:** Comprehensive Docker and Docker Compose validation following best practices and security standards

## Overview

The Docker Configuration Validator skill provides expert-level validation of Dockerfiles and Docker Compose files, ensuring your Docker configurations follow modern best practices, security standards, and syntax requirements. This skill automates the validation process, identifies issues, and provides specific, actionable fixes.

## Features

### Dockerfile Validation
- ✅ **Syntax Validation**: Hadolint integration for deep Dockerfile analysis
- ✅ **Multi-Stage Build Verification**: Ensures proper multi-stage implementation
- ✅ **Security Audit**: Checks for root users, exposed secrets, vulnerable images
- ✅ **Best Practices**: Enforces Docker best practices and optimization patterns
- ✅ **Layer Optimization**: Identifies inefficient layer structure

### Docker Compose Validation
- ✅ **Modern Syntax Compliance**: Ensures no obsolete `version` field (Compose v2.27.0+)
- ✅ **Image Tag Validation**: Checks for `:latest` tags and unpinned versions
- ✅ **Service Configuration**: Validates restart policies, health checks, resource limits
- ✅ **Network & Volume Best Practices**: Ensures proper networking and persistence
- ✅ **Security Review**: Checks for hardcoded secrets, privilege escalation

### Automation & Integration
- ✅ **Validation Scripts**: Generates custom bash scripts for automated validation
- ✅ **CI/CD Integration**: Provides GitHub Actions and GitLab CI configurations
- ✅ **Pre-commit Hooks**: Creates hooks for local validation before commits
- ✅ **Comprehensive Reports**: Detailed validation reports with prioritized issues

## When to Use This Skill

Use this skill when you need to:
- Validate Dockerfiles before committing to version control
- Review Docker configurations for production readiness
- Audit Docker security and identify vulnerabilities
- Ensure compliance with Docker best practices
- Modernize Docker Compose files (remove obsolete syntax)
- Create automated validation workflows
- Set up CI/CD pipeline validation
- Generate validation reports for team review
- Fix Docker configuration issues

## How It Works

### 1. Initial Assessment
The skill starts by understanding what you need validated:
- Which files to check (Dockerfile, Compose, or both)
- Specific concerns or focus areas
- Whether automation is needed

### 2. Comprehensive Validation
Performs multi-layered validation:
- **Syntax validation** using Hadolint and DCLint (if available)
- **Manual validation** covering all critical best practices
- **Security audit** for vulnerabilities and misconfigurations
- **Multi-stage build analysis** for optimization opportunities
- **Modern syntax compliance** (especially for Compose files)

### 3. Issue Classification
Categorizes findings by severity:
- **CRITICAL**: Must fix before deployment (security, functionality)
- **HIGH**: Should fix before production (reliability, best practices)
- **MEDIUM**: Recommended improvements (optimization, maintainability)
- **LOW**: Nice-to-have enhancements (polish, edge cases)

### 4. Detailed Reporting
Generates comprehensive reports including:
- Executive summary with overall status
- Specific issues with line numbers
- Before/after code examples
- Impact assessment for each issue
- Prioritized recommendations
- Effort estimates for fixes

### 5. Automation (Optional)
Can create:
- Custom validation scripts
- CI/CD pipeline configurations
- Pre-commit hooks
- Regular security scan setups

## Prerequisites

### Required
- **Docker**: Validation needs Docker CLI for syntax checking
  ```bash
  docker --version  # Should be 20.10+
  docker compose version  # Should be v2.27.0+
  ```

### Recommended
- **Hadolint**: Dockerfile linter (highly recommended)
  ```bash
  # macOS
  brew install hadolint

  # Linux
  wget -O /usr/local/bin/hadolint https://github.com/hadolint/hadolint/releases/download/v2.12.0/hadolint-Linux-x86_64
  chmod +x /usr/local/bin/hadolint

  # Verify
  hadolint --version
  ```

- **DCLint**: Docker Compose linter (recommended)
  ```bash
  # Install via npm
  npm install -g docker-compose-linter

  # Verify
  dclint --version
  ```

**Note:** The skill can perform validation even without these tools, but they significantly enhance the validation quality.

## Example Usage

### Basic Validation
```
User: "Can you validate my Dockerfile?"

Claude: "I'll validate your Dockerfile for best practices and security. Let me check..."

[Claude performs comprehensive validation and provides detailed report]
```

### Docker Compose Modernization
```
User: "Check my docker-compose.yml for issues"

Claude: "I'll validate your Docker Compose file. Checking for modern syntax compliance..."

[Claude identifies obsolete version field and other issues]

"❌ CRITICAL: Found obsolete 'version' field. This is deprecated in Compose v2.27.0+.
Remove the 'version: 3.8' line from your compose file."
```

### Full Project Audit
```
User: "Audit all Docker configs in my project for production readiness"

Claude: "I'll perform a comprehensive audit of all Dockerfiles and Compose files..."

[Claude validates all files and generates full security and best practices report]
```

### Automation Setup
```
User: "Create a validation script I can run in CI/CD"

Claude: "I'll create a comprehensive validation script with CI/CD integration..."

[Claude generates bash script + GitHub Actions workflow]
```

## Key Validation Checks

### Dockerfile Checks
- ❌ No `:latest` tags → Pin specific versions
- ❌ Running as root → Use non-root USER
- ❌ Single-stage build → Implement multi-stage
- ❌ Using ADD → Prefer COPY unless extracting
- ❌ No HEALTHCHECK → Add health monitoring
- ❌ No cache cleanup → Clean package manager caches
- ❌ Relative WORKDIR → Use absolute paths

### Docker Compose Checks
- ❌ `version:` field present → REMOVE (obsolete)
- ❌ `:latest` tags → Pin specific versions
- ❌ No restart policy → Add restart directives
- ❌ No health checks → Configure health monitoring
- ❌ No named volumes → Use named volumes for persistence
- ❌ No custom networks → Define custom networks
- ❌ Hardcoded secrets → Use environment variables or secrets management

### Security Checks
- ❌ Running as root user
- ❌ Secrets in image layers
- ❌ Unnecessary privileges
- ❌ Vulnerable base images
- ❌ Excessive port exposure
- ❌ Missing security updates

## Validation Report Example

```markdown
# Docker Configuration Validation Report

## Executive Summary
- Total Dockerfiles: 3
- Total Compose Files: 1
- Critical Issues: 2
- High Priority: 4
- Status: ⚠️ REQUIRES FIXES

## Critical Issues
1. Dockerfile (line 1): Using :latest tag
   Fix: FROM node:20-bullseye
   Impact: Unpredictable builds

2. docker-compose.yml (line 1): Obsolete version field
   Fix: Remove 'version: 3.8' line
   Impact: Using deprecated syntax

## Recommendations
- Immediate: Pin all image versions, remove version field
- High Priority: Add health checks, configure restart policies
- Medium: Implement multi-stage builds for optimization
```

## Common Issues & Fixes

### Issue: Obsolete Docker Compose Version Field
**Problem:**
```yaml
version: '3.8'  # ❌ Obsolete!
services:
  web:
    image: nginx
```

**Fix:**
```yaml
# No version field needed!
services:
  web:
    image: nginx:1.24-alpine
```

### Issue: Running as Root
**Problem:**
```dockerfile
FROM alpine
COPY app /app
CMD ["/app/server"]  # Runs as root!
```

**Fix:**
```dockerfile
FROM alpine
RUN addgroup -g 1001 appuser && \
    adduser -D -u 1001 -G appuser appuser
COPY --chown=appuser:appuser app /app
USER appuser
CMD ["/app/server"]
```

### Issue: No Multi-Stage Build
**Problem:**
```dockerfile
FROM node:18
COPY . .
RUN npm install  # Includes dev dependencies!
CMD ["npm", "start"]
```

**Fix:**
```dockerfile
# Build stage
FROM node:18 AS builder
COPY . .
RUN npm ci && npm run build

# Runtime stage
FROM node:18-alpine
COPY --from=builder /app/dist ./dist
RUN npm ci --only=production
CMD ["node", "dist/index.js"]
```

## Best Practices Enforced

### Dockerfile Best Practices
- Use specific image tags (no `:latest`)
- Implement multi-stage builds
- Run as non-root user
- Use COPY instead of ADD
- Add HEALTHCHECK instructions
- Clean package manager caches
- Use absolute WORKDIR paths
- Add appropriate labels

### Docker Compose Best Practices
- No `version` field (obsolete in v2.27.0+)
- Pin all image versions
- Define restart policies
- Configure health checks
- Use named volumes
- Define custom networks
- Manage secrets properly
- Set resource limits

### Security Best Practices
- Never run as root
- No secrets in images
- Use minimal base images
- Regular security scans
- Least privilege principle
- Network isolation
- Read-only root filesystem (where applicable)

## Integration Examples

### GitHub Actions
```yaml
name: Docker Validation
on: [push, pull_request]

jobs:
  validate:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: hadolint/hadolint-action@v3.1.0
        with:
          dockerfile: Dockerfile
      - run: docker compose config --quiet
```

### GitLab CI
```yaml
docker-validation:
  stage: validate
  image: hadolint/hadolint:latest
  script:
    - hadolint Dockerfile
    - docker compose config --quiet
```

### Pre-commit Hook
```bash
#!/bin/bash
# .git/hooks/pre-commit
if git diff --cached --name-only | grep -q "Dockerfile"; then
    hadolint Dockerfile || exit 1
fi
```

## Resources

### Validation Tools
- [Hadolint](https://github.com/hadolint/hadolint) - Dockerfile linter
- [DCLint](https://github.com/zavoloklom/docker-compose-linter) - Compose linter
- [Trivy](https://github.com/aquasecurity/trivy) - Security scanner

### Documentation
- [Docker Best Practices](https://docs.docker.com/develop/dev-best-practices/)
- [Dockerfile Reference](https://docs.docker.com/engine/reference/builder/)
- [Compose Specification](https://docs.docker.com/compose/compose-file/)
- [Hadolint Rules](https://github.com/hadolint/hadolint#rules)

### Learning Resources
- [Multi-Stage Builds Guide](https://docs.docker.com/build/building/multi-stage/)
- [Security Best Practices](https://docs.docker.com/engine/security/)
- [Compose v2 Migration](https://docs.docker.com/compose/migrate/)

## Support

For issues with this skill or suggestions for improvements:
- Check the skill's SKILL.md for detailed workflow
- Review validation-checklist.md for complete rule set
- Consult tool-installation.md for setup help

## Version History

### 1.0.0 (2025-10-18)
- Initial release
- Comprehensive Dockerfile validation
- Docker Compose validation with modern syntax checking
- Multi-stage build verification
- Security audit capabilities
- Automation script generation
- CI/CD integration templates

---

**Maintainer:** rknall
**Last Updated:** 2025-10-18
**Skill Type:** Development / DevOps
