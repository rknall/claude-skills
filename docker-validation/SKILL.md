---
name: "Docker Configuration Validator"
description: "Comprehensive Docker and Docker Compose validation following best practices and security standards. Use this skill when users ask to validate Dockerfiles, review Docker configurations, check Docker Compose files, verify multi-stage builds, audit Docker security, or ensure compliance with Docker best practices. Validates syntax, security, multi-stage builds, and modern Docker Compose requirements."
---

# Docker Configuration Validator

This skill provides comprehensive validation for Dockerfiles and Docker Compose files, ensuring compliance with best practices, security standards, and modern syntax requirements.

## When to Use This Skill

Activate this skill when the user requests:
- Validate Dockerfiles or Docker Compose files
- Review Docker configurations for best practices
- Check for Docker security issues
- Verify multi-stage build implementation
- Audit Docker setup for production readiness
- Ensure modern Docker Compose syntax compliance
- Fix Docker configuration issues
- Create validation scripts or CI/CD integration

## Core Workflow

### Phase 1: Initial Assessment

When a user requests Docker validation, start by understanding the scope:

1. **Identify What to Validate**
   - Single Dockerfile or multiple?
   - Docker Compose file(s)?
   - Entire project directory?
   - Specific concerns or focus areas?
   - Production vs development environment?

2. **Understand Requirements**
   - Security compliance level needed?
   - Multi-stage build requirement?
   - Performance constraints?
   - CI/CD integration needed?
   - Automated validation script desired?

3. **Check Available Tools**
   - Is Hadolint available? (Dockerfile linter)
   - Is DCLint available? (Docker Compose linter)
   - Is Docker CLI available?
   - Should tools be installed as part of process?

### Phase 2: Dockerfile Validation

#### Step 1: Locate Dockerfiles

Find all Dockerfiles in the project:
```bash
find . -type f \( -name "Dockerfile*" ! -name "*.md" \)
```

#### Step 2: Run Hadolint Validation

**If Hadolint is available:**
```bash
# Validate each Dockerfile
hadolint Dockerfile

# JSON output for programmatic analysis
hadolint --format json Dockerfile

# Set failure threshold
hadolint --failure-threshold error Dockerfile
```

**If Hadolint is NOT available:**
- Perform manual validation using built-in checks (see Step 3)
- Recommend Hadolint installation
- Provide installation instructions

#### Step 3: Manual Dockerfile Validation

Check each Dockerfile for critical issues:

**1. Base Image Best Practices**
- ✅ Check: No `:latest` tags
- ✅ Check: Specific version pinned
- ✅ Check: Uses minimal base images (alpine/slim variants)
- ✅ Check: Trusted registry used

**2. Multi-Stage Build Verification**
```bash
# Count FROM statements (should be >= 2 for multi-stage)
FROM_COUNT=$(grep -c "^FROM " Dockerfile)

# Check for named stages
NAMED_STAGES=$(grep -c "^FROM .* AS " Dockerfile)

# Check for inter-stage copies
COPY_FROM=$(grep -c "COPY --from=" Dockerfile)
```

**Analysis:**
- If FROM_COUNT >= 2: Multi-stage build detected ✅
- If COPY_FROM == 0 and FROM_COUNT >= 2: Warning - artifacts may not be transferred
- If FROM_COUNT == 1: Single-stage build - recommend multi-stage for optimization

**3. Security Checks**
- ✅ Check: USER directive present (non-root)
- ✅ Check: Last USER is not root
- ✅ Check: No secrets in Dockerfile
- ✅ Check: WORKDIR uses absolute paths
- ✅ Check: No unnecessary privileges

**4. Layer Optimization**
- ✅ Check: Combined RUN commands
- ✅ Check: Package manager cache cleaned
- ✅ Check: COPY before RUN for better caching
- ✅ Check: .dockerignore file exists

**5. Best Practices**
- ✅ Check: HEALTHCHECK defined
- ✅ Check: WORKDIR used (not cd)
- ✅ Check: COPY preferred over ADD
- ✅ Check: CMD/ENTRYPOINT uses JSON notation
- ✅ Check: Labels included (maintainer, version)

#### Step 4: Dockerfile Issue Classification

Classify findings by severity:

**CRITICAL (Must Fix):**
- Running as root user
- Using `:latest` tags
- Secrets exposed in image
- Invalid syntax

**HIGH (Should Fix):**
- No multi-stage build when applicable
- No HEALTHCHECK
- Package versions not pinned
- Security vulnerabilities

**MEDIUM (Recommended):**
- Using ADD instead of COPY
- Not cleaning package cache
- Missing labels
- Inefficient layer structure

**LOW (Nice to Have):**
- Could use slimmer base image
- Could combine more RUN commands
- Could improve comments

### Phase 3: Docker Compose Validation

#### Step 1: Locate Compose Files

Find all Docker Compose files:
```bash
find . -maxdepth 3 -type f \( -name "docker-compose*.yml" -o -name "docker-compose*.yaml" -o -name "compose*.yml" \)
```

#### Step 2: Check for Obsolete Version Field

**CRITICAL CHECK:** Modern Docker Compose (v2.27.0+) does NOT use version field

```bash
# Check for obsolete version field
if grep -q "^version:" docker-compose.yml; then
    echo "❌ ERROR: Found obsolete 'version' field"
    echo "Remove 'version:' line - it's obsolete in Compose v2.27.0+"
fi
```

**Old (Deprecated):**
```yaml
version: '3.8'  # ❌ REMOVE THIS
services:
  web:
    image: nginx:latest
```

**New (Modern):**
```yaml
# No version field!
services:
  web:
    image: nginx:1.24-alpine
```

#### Step 3: Built-in Docker Compose Validation

```bash
# Syntax validation
docker compose config --quiet

# Show resolved configuration
docker compose config

# Validate specific file
docker compose -f docker-compose.prod.yml config --quiet
```

#### Step 4: DCLint Validation (if available)

```bash
# Lint compose file
dclint docker-compose.yml

# Auto-fix issues
dclint --fix docker-compose.yml

# JSON output
dclint --format json docker-compose.yml
```

#### Step 5: Manual Compose Validation

**1. Image Best Practices**
- ✅ Check: No `:latest` tags
- ✅ Check: Specific versions specified
- ✅ Check: Images from trusted sources

**2. Service Configuration**
- ✅ Check: Restart policies defined
- ✅ Check: Health checks configured
- ✅ Check: Resource limits set (optional but recommended)
- ✅ Check: Proper service dependencies (depends_on)

**3. Networking**
- ✅ Check: Custom networks defined
- ✅ Check: Network isolation implemented
- ✅ Check: Appropriate network drivers used

**4. Volumes & Persistence**
- ✅ Check: Named volumes for persistence
- ✅ Check: Volume drivers specified
- ✅ Check: Bind mounts use absolute paths
- ✅ Check: No sensitive data in volumes

**5. Environment & Secrets**
- ✅ Check: Environment variables properly managed
- ✅ Check: No hardcoded secrets
- ✅ Check: .env file usage recommended
- ✅ Check: Secrets management configured

**6. Security**
- ✅ Check: No privileged mode (unless necessary)
- ✅ Check: Capabilities properly configured
- ✅ Check: User/group specified
- ✅ Check: Read-only root filesystem (where applicable)

### Phase 4: Multi-Stage Build Deep Dive

When validating multi-stage builds, ensure:

**1. Stage Structure**
```dockerfile
# Build stage
FROM node:20-bullseye AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build && npm run test

# Production stage
FROM node:20-alpine AS runtime
WORKDIR /app
COPY --from=builder /app/dist ./dist
USER node
CMD ["node", "dist/index.js"]
```

**2. Validation Checklist**
- ✅ At least 2 stages (build + runtime)
- ✅ All stages are named with AS keyword
- ✅ Artifacts copied between stages using COPY --from
- ✅ Final stage uses minimal base image
- ✅ Build tools NOT in final stage
- ✅ Only necessary files in final image
- ✅ Final stage runs as non-root user

**3. Common Patterns to Check**

**Node.js Multi-Stage:**
- Build stage: Install all deps, build, test
- Runtime stage: Production deps only, built artifacts

**Python Multi-Stage:**
- Build stage: Compile dependencies, build wheels
- Runtime stage: Install pre-built wheels, app only

**Go Multi-Stage:**
- Build stage: Full Go toolchain, compile binary
- Runtime stage: Scratch/distroless, binary only

### Phase 5: Security Audit

Perform comprehensive security checks:

**1. User & Permissions**
```bash
# Check for USER directive
USER_COUNT=$(grep -c "^USER " Dockerfile)
LAST_USER=$(grep "^USER " Dockerfile | tail -1 | awk '{print $2}')

if [ "$USER_COUNT" -eq 0 ]; then
    echo "❌ CRITICAL: No USER specified (runs as root!)"
elif [ "$LAST_USER" == "root" ] || [ "$LAST_USER" == "0" ]; then
    echo "❌ CRITICAL: Final USER is root"
fi
```

**2. Exposed Secrets**
- Check for passwords in ENV
- Check for API keys in COPY/ADD
- Check for credentials in RUN commands
- Recommend secrets management solutions

**3. Vulnerable Base Images**
- Check base image age and support status
- Recommend security scanning (Trivy, Snyk)
- Suggest pinned, supported versions

**4. Network Exposure**
- Review EXPOSE directives
- Check if unnecessary ports exposed
- Validate port mappings in Compose

### Phase 6: Generate Validation Report

Structure the validation report:

```markdown
# Docker Configuration Validation Report

## Executive Summary
- **Total Dockerfiles Analyzed**: X
- **Total Compose Files Analyzed**: X
- **Critical Issues**: X
- **High Priority Issues**: X
- **Medium Priority Issues**: X
- **Low Priority Issues**: X
- **Overall Status**: ✅ PASS / ⚠️ WARNINGS / ❌ FAIL

## Dockerfile Analysis

### [Dockerfile Path]

#### Validation Results
✅ **PASSED**: Multi-stage build implemented
✅ **PASSED**: Running as non-root user
⚠️  **WARNING**: Using ADD instead of COPY (line 15)
❌ **FAILED**: Base image uses :latest tag (line 1)

#### Security Assessment
- **User**: nodejs (non-root) ✅
- **Base Image**: node:latest ❌
- **Secrets**: None detected ✅
- **Health Check**: Present ✅

#### Multi-Stage Build Analysis
- **Stages**: 2 (builder, runtime)
- **Named Stages**: Yes ✅
- **Inter-Stage Copies**: 1 ✅
- **Final Base**: node:20-alpine ✅
- **Build Tools in Final**: No ✅

#### Issues Found

**CRITICAL:**
1. Line 1: Using :latest tag
   - Issue: `FROM node:latest`
   - Fix: `FROM node:20-bullseye`
   - Impact: Unpredictable builds, security risk

**HIGH:**
2. Line 45: Package cache not cleaned
   - Issue: `RUN apt-get install -y curl`
   - Fix: Add `&& rm -rf /var/lib/apt/lists/*`
   - Impact: Larger image size

**MEDIUM:**
3. Line 15: Using ADD instead of COPY
   - Issue: `ADD app.tar.gz /app/`
   - Fix: `COPY app.tar.gz /app/` (or extract separately)
   - Impact: Less explicit behavior

## Docker Compose Analysis

### [Compose File Path]

#### Validation Results
❌ **FAILED**: Obsolete 'version' field present
⚠️  **WARNING**: 3 services using :latest tags
✅ **PASSED**: Custom networks defined
✅ **PASSED**: Named volumes configured

#### Modern Syntax Compliance
- **Version Field**: Present ❌ (MUST REMOVE)
- **Specific Tags**: 2 of 5 services ⚠️
- **Restart Policies**: All services ✅
- **Health Checks**: 3 of 5 services ⚠️

#### Issues Found

**CRITICAL:**
1. Obsolete 'version' field
   - Issue: `version: '3.8'`
   - Fix: Remove the entire line
   - Impact: Using deprecated syntax

**HIGH:**
2. Service 'web' uses :latest tag
   - Issue: `image: nginx:latest`
   - Fix: `image: nginx:1.24-alpine`
   - Impact: Unpredictable deployments

3. No restart policy on 'cache' service
   - Issue: Missing `restart:` directive
   - Fix: Add `restart: unless-stopped`
   - Impact: Service won't auto-recover

## Recommendations

### Immediate Actions (Critical)
1. Pin all Docker image versions (remove :latest)
2. Remove obsolete 'version' field from Compose files
3. Ensure all services run as non-root users
4. Clean package manager caches in all Dockerfiles

### High Priority (Before Production)
1. Implement health checks for all services
2. Add restart policies to all Compose services
3. Set up resource limits (memory, CPU)
4. Implement proper secrets management

### Medium Priority (Best Practices)
1. Use COPY instead of ADD unless extracting archives
2. Add labels for better organization
3. Implement logging drivers
4. Add metadata to images

### Low Priority (Optimizations)
1. Consider slimmer base images where possible
2. Optimize layer caching with better ordering
3. Add comprehensive comments
4. Implement .dockerignore files

## Next Steps

1. **Fix Critical Issues** (Estimated: 30 minutes)
   - Pin image versions
   - Remove version field from Compose
   - Add USER directives

2. **Address High Priority** (Estimated: 1-2 hours)
   - Add health checks
   - Configure restart policies
   - Set resource limits

3. **Re-validate** (Estimated: 10 minutes)
   - Run validation again after fixes
   - Ensure all critical issues resolved

4. **Set Up Automation** (Estimated: 1 hour)
   - Add pre-commit hooks
   - Integrate into CI/CD
   - Schedule regular security scans

## Validation Tools Recommendations

### Install Required Tools
```bash
# Hadolint (Dockerfile linter)
brew install hadolint  # macOS
# or
wget -O /usr/local/bin/hadolint https://github.com/hadolint/hadolint/releases/download/v2.12.0/hadolint-Linux-x86_64

# DCLint (Docker Compose linter)
npm install -g docker-compose-linter

# Trivy (Security scanner)
brew install aquasecurity/trivy/trivy
```

### Automated Validation Script
[Provide custom validation script based on project needs]

### CI/CD Integration
[Provide GitHub Actions / GitLab CI configuration]
```

## Deliverables

At the end of validation, provide:

1. **Comprehensive Validation Report**
   - Executive summary with overall status
   - Detailed findings per file
   - Issue classification (Critical → Low)
   - Specific fixes for each issue
   - Recommendations prioritized

2. **Fixed Configuration Files** (if requested)
   - Corrected Dockerfiles
   - Updated Compose files
   - .hadolint.yaml configuration
   - .dclintrc.json configuration

3. **Validation Script** (if requested)
   - Custom bash script for automated validation
   - Checks for project-specific requirements
   - Color-coded output
   - Exit codes for CI/CD

4. **CI/CD Integration** (if requested)
   - GitHub Actions workflow
   - GitLab CI configuration
   - Pre-commit hooks
   - Integration with existing pipeline

5. **Documentation**
   - How to run validation locally
   - Tool installation instructions
   - Best practices guide
   - Common issues and solutions

## Validation Script Template

When user requests automated validation, create this script:

```bash
#!/bin/bash
# docker-validation.sh

set -e

# Color codes
RED='\033[0;31m'
GREEN='\033[0;32m'
YELLOW='\033[1;33m'
BLUE='\033[0;34m'
NC='\033[0m'

ERRORS=0
WARNINGS=0

echo -e "${BLUE}╔══════════════════════════════════════════╗${NC}"
echo -e "${BLUE}║   Docker Configuration Validator        ║${NC}"
echo -e "${BLUE}╚══════════════════════════════════════════╝${NC}"

# 1. Prerequisites check
echo -e "\n${BLUE}━━━ Checking Prerequisites ━━━${NC}"
command -v docker >/dev/null || { echo "❌ Docker not found"; exit 1; }
command -v hadolint >/dev/null || echo "⚠️  Hadolint not installed"

# 2. Dockerfile validation
echo -e "\n${BLUE}━━━ Validating Dockerfiles ━━━${NC}"
find . -name "Dockerfile*" -type f | while read df; do
    echo "Checking: $df"
    # [Validation logic here]
done

# 3. Multi-stage build check
echo -e "\n${BLUE}━━━ Checking Multi-Stage Builds ━━━${NC}"
# [Multi-stage validation logic]

# 4. Compose validation
echo -e "\n${BLUE}━━━ Validating Docker Compose ━━━${NC}"
find . -name "*compose*.yml" | while read cf; do
    # Check for obsolete version field
    if grep -q "^version:" "$cf"; then
        echo -e "${RED}❌ $cf: Obsolete version field${NC}"
        ERRORS=$((ERRORS + 1))
    fi

    # Validate syntax
    docker compose -f "$cf" config --quiet || ERRORS=$((ERRORS + 1))
done

# 5. Security checks
echo -e "\n${BLUE}━━━ Security Audit ━━━${NC}"
# [Security validation logic]

# Final report
echo -e "\n${BLUE}╔══════════════════════════════════════════╗${NC}"
echo -e "${BLUE}║         Validation Summary               ║${NC}"
echo -e "${BLUE}╚══════════════════════════════════════════╝${NC}"

if [ $ERRORS -eq 0 ]; then
    echo -e "${GREEN}✅ All checks passed!${NC}"
    exit 0
else
    echo -e "${RED}❌ Found $ERRORS error(s), $WARNINGS warning(s)${NC}"
    exit 1
fi
```

## CI/CD Integration Templates

### GitHub Actions

```yaml
name: Docker Validation

on:
  push:
    branches: [ main, develop ]
  pull_request:
    branches: [ main ]

jobs:
  validate:
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v4

      - name: Lint Dockerfiles
        uses: hadolint/hadolint-action@v3.1.0
        with:
          dockerfile: ./Dockerfile
          failure-threshold: error

      - name: Validate Compose files
        run: |
          for f in $(find . -name "*compose*.yml"); do
            docker compose -f "$f" config --quiet
          done

      - name: Check for obsolete version field
        run: |
          if grep -r "^version:" . --include="*compose*.yml"; then
            echo "ERROR: Found obsolete version field"
            exit 1
          fi
```

### GitLab CI

```yaml
stages:
  - validate

docker-validation:
  stage: validate
  image: hadolint/hadolint:latest-debian
  script:
    - find . -name "Dockerfile*" -exec hadolint {} \;
    - |
      for f in $(find . -name "*compose*.yml"); do
        if grep -q "^version:" "$f"; then
          echo "ERROR: Obsolete version field in $f"
          exit 1
        fi
      done
```

## Communication Style

When conducting validation:

1. **Be Thorough**
   - Check all aspects systematically
   - Don't skip manual checks if tools unavailable
   - Validate both syntax and best practices

2. **Prioritize Clearly**
   - Critical issues first (security, functionality)
   - High priority next (production readiness)
   - Medium and low as improvements

3. **Provide Specific Fixes**
   - Show exact line numbers
   - Provide before/after code
   - Explain why the fix is needed
   - Include impact assessment

4. **Educate**
   - Explain Docker best practices
   - Reference official documentation
   - Provide learning resources
   - Share modern syntax requirements

5. **Be Actionable**
   - Give step-by-step remediation
   - Provide automation scripts
   - Suggest CI/CD integration
   - Estimate effort for fixes

## Common Validation Patterns

### Pattern 1: Quick Syntax Check
- Run Hadolint and DCLint
- Check for obsolete version field
- Validate with Docker Compose config
- Report critical syntax errors only

### Pattern 2: Full Production Audit
- Complete Dockerfile validation
- Multi-stage build verification
- Security audit
- Compose best practices
- Generate comprehensive report
- Provide automation scripts

### Pattern 3: Security-Focused Review
- User privilege checks
- Secret exposure scan
- Base image security
- Network exposure audit
- Security scanning integration

### Pattern 4: Optimization Review
- Multi-stage build effectiveness
- Layer optimization
- Image size analysis
- Build time improvements
- Caching strategies

## Key Rules to Enforce

### Dockerfile Rules (Critical)
- ✅ No `:latest` tags (DL3006)
- ✅ Last USER not root (DL3002)
- ✅ Use absolute WORKDIR (DL3000)
- ✅ Pin package versions (DL3008, DL3013)
- ✅ Clean package cache (DL3009)

### Docker Compose Rules (Critical)
- ✅ NO version field (obsolete since v2.27.0)
- ✅ No `:latest` tags
- ✅ Restart policies defined
- ✅ Health checks configured
- ✅ Named volumes for persistence

### Multi-Stage Build Rules
- ✅ Minimum 2 stages
- ✅ Named stages with AS
- ✅ COPY --from for artifacts
- ✅ Minimal final base image
- ✅ No build tools in final stage

### Security Rules (Critical)
- ✅ Run as non-root USER
- ✅ No secrets in images
- ✅ Minimal base images
- ✅ Regular security scans
- ✅ Least privilege principle

## Reference Resources

When providing recommendations, reference:
- **Docker Best Practices**: Official Docker documentation
- **Hadolint Rules**: Complete rule reference
- **Compose Specification**: Latest Compose spec
- **Security Standards**: CIS Docker Benchmark
- **Multi-Stage Builds**: Official guide and examples

Remember: Docker validation is about ensuring reliable, secure, and efficient containerized applications. Always prioritize security and production readiness while maintaining developer experience.
