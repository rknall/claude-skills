# Docker Validation Guide 🐳

## Table of Contents
- [Overview](#overview)
- [Prerequisites](#prerequisites)
- [Dockerfile Validation](#dockerfile-validation)
- [Docker Compose Validation](#docker-compose-validation)
- [Multi-Stage Build Verification](#multi-stage-build-verification)
- [Automated Validation Script](#automated-validation-script)
- [CI/CD Integration](#cicd-integration)
- [Common Issues & Solutions](#common-issues--solutions)
- [Best Practices Checklist](#best-practices-checklist)

---

## Overview

This guide provides a comprehensive approach to validating Docker installations, Dockerfiles, and Docker Compose files. It ensures your Docker configurations follow best practices, security standards, and modern syntax requirements.

### What This Guide Covers
- ✅ Dockerfile linting and validation
- ✅ Docker Compose file validation
- ✅ Multi-stage build verification
- ✅ Security best practices
- ✅ Modern syntax compliance
- ✅ Automated validation workflows
- ✅ CI/CD pipeline integration

---

## Prerequisites

### Required Tools

#### 1. Hadolint (Dockerfile Linter)
**Installation:**

```bash
# macOS
brew install hadolint

# Linux (download binary)
wget -O /usr/local/bin/hadolint https://github.com/hadolint/hadolint/releases/download/v2.12.0/hadolint-Linux-x86_64
chmod +x /usr/local/bin/hadolint

# Using Docker
docker pull hadolint/hadolint:latest

# Verify installation
hadolint --version
```

**Why Hadolint?**
- Parses Dockerfile into AST for deep analysis
- Integrates ShellCheck for bash validation
- Enforces Docker best practices
- Customizable rules via `.hadolint.yaml`

#### 2. Docker Compose Linter (DCLint)
**Installation:**

```bash
# Install via npm (requires Node.js 20.19.0+)
npm install -g docker-compose-linter

# Or use npx without global install
npx dclint --version

# Verify installation
dclint --version
```

#### 3. Docker CLI
**Verify Docker Installation:**

```bash
# Check Docker version
docker --version

# Check Docker Compose version
docker compose version

# Verify Docker is running
docker ps
```

---

## Dockerfile Validation

### Basic Validation with Hadolint

```bash
# Validate single Dockerfile
hadolint Dockerfile

# Validate with specific rules ignored
hadolint --ignore DL3003 --ignore DL3006 Dockerfile

# Validate multiple Dockerfiles
find . -name "Dockerfile*" -exec hadolint {} \;

# Output in JSON format for parsing
hadolint --format json Dockerfile

# Set failure threshold (only fail on errors, not warnings)
hadolint --failure-threshold error Dockerfile
```

### Hadolint Configuration File

Create `.hadolint.yaml` in your project root:

```yaml
# .hadolint.yaml
---
# Ignore specific rules globally
ignored:
  - DL3008  # Pin versions in apt-get install (sometimes too strict)
  - DL3013  # Pin versions in pip install
  - DL3018  # Pin versions in apk add

# Override rule severity
override:
  error:
    - DL3001  # Use absolute WORKDIR (critical)
    - DL3002  # Last user should not be root (security)
    - DL3025  # Use JSON notation for CMD (best practice)
  warning:
    - DL3003  # Use WORKDIR instead of cd
  info:
    - DL3006  # Always tag image explicitly

# Trusted registries (won't warn about these)
trustedRegistries:
  - docker.io
  - ghcr.io
  - gcr.io
  - quay.io
  - registry.hub.docker.com

# Enforce specific label schema
label-schema:
  maintainer: text
  org.opencontainers.image.title: text
  org.opencontainers.image.description: text
  org.opencontainers.image.version: text

# Require specific labels
required-labels:
  - maintainer
  - org.opencontainers.image.version

# Enable strict label validation
strict-labels: true
```

### Inline Rule Suppression

```dockerfile
# Ignore specific rule for next line
# hadolint ignore=DL3003,DL3006
RUN cd /tmp && wget https://example.com/file.tar.gz

# Ignore rule for a specific instruction
RUN apt-get update && \
    # hadolint ignore=DL3008,DL3015
    apt-get install -y python3-pip && \
    rm -rf /var/lib/apt/lists/*
```

### Key Dockerfile Rules to Check

| Rule | Description | Severity |
|------|-------------|----------|
| DL3000 | Use absolute WORKDIR | Error |
| DL3001 | Switch to non-root USER | Error |
| DL3002 | Last USER should not be root | Error |
| DL3003 | Use WORKDIR, not cd | Warning |
| DL3006 | Always tag image explicitly | Warning |
| DL3008 | Pin versions in apt-get | Warning |
| DL3009 | Delete apt-get lists after install | Info |
| DL3013 | Pin versions in pip | Warning |
| DL3020 | Use COPY instead of ADD | Warning |
| DL3025 | Use JSON notation for CMD/ENTRYPOINT | Warning |

---

## Docker Compose Validation

### Built-in Docker Compose Validation

```bash
# Basic validation (checks syntax and structure)
docker compose config

# Quiet mode (only shows errors)
docker compose config --quiet

# Validate specific file
docker compose -f docker-compose.prod.yml config --quiet

# Show resolved configuration
docker compose config --resolve-image-digests

# Validate and output to file
docker compose config > resolved-compose.yml
```

### DCLint - Advanced Compose Linting

```bash
# Lint current directory (auto-finds compose files)
dclint

# Lint specific file
dclint docker-compose.yml

# Lint with auto-fix
dclint --fix docker-compose.yml

# Use custom configuration
dclint --config .dclint.json docker-compose.yml

# Output format options
dclint --format json docker-compose.yml
dclint --format table docker-compose.yml
```

### DCLint Configuration

Create `.dclintrc.json`:

```json
{
  "rules": {
    "no-version-field": "error",
    "require-quotes": "warning",
    "service-name-case": ["error", "kebab-case"],
    "alphabetical-keys": "warning",
    "no-duplicate-keys": "error",
    "require-restart": "warning"
  },
  "exclude": [
    "node_modules/**",
    ".git/**",
    "dist/**"
  ],
  "sort-keys": true,
  "fix": false
}
```

### Modern Docker Compose Syntax Requirements

**❌ OLD (Deprecated):**
```yaml
version: '3.8'  # ⚠️ Version field is obsolete!

services:
  web:
    image: nginx:latest
    ports:
      - "80:80"
```

**✅ NEW (Modern Syntax):**
```yaml
# No version field needed!
# Docker Compose uses latest specification automatically

services:
  web:
    image: nginx:1.24-alpine  # Always use specific tags
    ports:
      - "80:80"
    restart: unless-stopped
    networks:
      - frontend
    labels:
      - "com.example.description=Web server"

networks:
  frontend:
    driver: bridge
```

### Key Docker Compose Checks

- ✅ **No `version` field** (obsolete since Compose v2.27.0)
- ✅ **Specific image tags** (avoid `:latest`)
- ✅ **Named volumes** for persistence
- ✅ **Health checks** for critical services
- ✅ **Resource limits** defined
- ✅ **Restart policies** specified
- ✅ **Networks** properly configured
- ✅ **Environment variables** properly managed

---

## Multi-Stage Build Verification

### What to Check for Multi-Stage Builds

A proper multi-stage Dockerfile should:
1. Have at least 2 stages (build + runtime)
2. Use named stages with `AS` keyword
3. Copy only necessary artifacts between stages
4. Use minimal base image for final stage
5. Not include build tools in final image

### Multi-Stage Build Validation Script

```bash
#!/bin/bash
# validate-multistage.sh

echo "🔍 Checking for Multi-Stage Build..."

DOCKERFILE="${1:-Dockerfile}"

if [ ! -f "$DOCKERFILE" ]; then
    echo "❌ Dockerfile not found: $DOCKERFILE"
    exit 1
fi

# Count FROM statements (should be >= 2 for multi-stage)
FROM_COUNT=$(grep -c "^FROM " "$DOCKERFILE")

if [ "$FROM_COUNT" -lt 2 ]; then
    echo "⚠️  WARNING: Only $FROM_COUNT FROM statement(s) found."
    echo "   Multi-stage builds should have at least 2 stages (build + runtime)."
else
    echo "✅ Found $FROM_COUNT stages (multi-stage build detected)"
fi

# Check if stages are named (best practice)
NAMED_STAGES=$(grep "^FROM .* AS " "$DOCKERFILE" | wc -l)
echo "ℹ️  Named stages found: $NAMED_STAGES"

if [ "$NAMED_STAGES" -gt 0 ] && [ "$NAMED_STAGES" -lt "$FROM_COUNT" ]; then
    echo "⚠️  WARNING: Not all stages are named. Consider naming all stages for clarity."
fi

# Check for COPY --from usage
COPY_FROM=$(grep -c "COPY --from=" "$DOCKERFILE")
if [ "$COPY_FROM" -gt 0 ]; then
    echo "✅ Found $COPY_FROM inter-stage COPY operations"
else
    echo "⚠️  WARNING: No COPY --from found. Are artifacts being copied between stages?"
fi

# Check final stage base image
FINAL_BASE=$(grep "^FROM " "$DOCKERFILE" | tail -1 | awk '{print $2}')
echo "ℹ️  Final stage base image: $FINAL_BASE"

# Suggest improvements for common base images
case $FINAL_BASE in
    *alpine*)
        echo "✅ Using Alpine base (good for minimal images)"
        ;;
    *slim*)
        echo "✅ Using slim variant (good compromise)"
        ;;
    *:latest)
        echo "⚠️  WARNING: Using :latest tag. Pin to specific version!"
        ;;
    ubuntu:*|debian:*)
        echo "⚠️  Consider using slim or alpine variants for smaller final image"
        ;;
esac

echo ""
echo "📊 Multi-Stage Build Analysis Complete!"
```

### Example: Proper Multi-Stage Dockerfile

```dockerfile
# syntax=docker/dockerfile:1

#
# Stage 1: Build Stage
#
FROM node:20-bullseye AS builder

WORKDIR /app

# Copy dependency files first (better caching)
COPY package*.json ./

# Install ALL dependencies (including dev dependencies)
RUN npm ci

# Copy application source
COPY . .

# Run build process
RUN npm run build && \
    npm run test

#
# Stage 2: Production Runtime Stage
#
FROM node:20-alpine AS runtime

# Create non-root user
RUN addgroup -g 1001 -S nodejs && \
    adduser -S nodejs -u 1001

WORKDIR /app

# Copy only production dependencies
COPY package*.json ./
RUN npm ci --only=production && \
    npm cache clean --force

# Copy built artifacts from builder stage
COPY --from=builder --chown=nodejs:nodejs /app/dist ./dist

# Switch to non-root user
USER nodejs

# Expose port
EXPOSE 3000

# Health check
HEALTHCHECK --interval=30s --timeout=3s --start-period=5s --retries=3 \
    CMD node -e "require('http').get('http://localhost:3000/health', (r) => process.exit(r.statusCode === 200 ? 0 : 1))"

# Start application
CMD ["node", "dist/index.js"]
```

---

## Automated Validation Script

### Complete Validation Script

Create `validate-docker.sh`:

```bash
#!/bin/bash
# validate-docker.sh - Comprehensive Docker validation script

set -e  # Exit on error

# Color codes for output
RED='\033[0;31m'
GREEN='\033[0;32m'
YELLOW='\033[1;33m'
BLUE='\033[0;34m'
NC='\033[0m' # No Color

# Counters
ERRORS=0
WARNINGS=0

echo -e "${BLUE}╔════════════════════════════════════════════════════════════╗${NC}"
echo -e "${BLUE}║         Docker Configuration Validation Suite             ║${NC}"
echo -e "${BLUE}╚════════════════════════════════════════════════════════════╝${NC}"
echo ""

#
# Function: Check if command exists
#
command_exists() {
    command -v "$1" >/dev/null 2>&1
}

#
# Function: Print section header
#
print_section() {
    echo -e "\n${BLUE}━━━ $1 ━━━${NC}"
}

#
# 1. PREREQUISITE CHECKS
#
print_section "1️⃣  Checking Prerequisites"

if ! command_exists docker; then
    echo -e "${RED}❌ Docker not found. Please install Docker.${NC}"
    exit 1
fi
echo -e "${GREEN}✅ Docker installed: $(docker --version)${NC}"

if ! command_exists hadolint; then
    echo -e "${YELLOW}⚠️  Hadolint not found. Install with: brew install hadolint${NC}"
    WARNINGS=$((WARNINGS + 1))
else
    echo -e "${GREEN}✅ Hadolint installed: $(hadolint --version)${NC}"
fi

if ! command_exists dclint; then
    echo -e "${YELLOW}⚠️  DCLint not found. Install with: npm install -g docker-compose-linter${NC}"
    WARNINGS=$((WARNINGS + 1))
else
    echo -e "${GREEN}✅ DCLint installed${NC}"
fi

#
# 2. DOCKERFILE VALIDATION
#
print_section "2️⃣  Validating Dockerfiles"

DOCKERFILES=$(find . -type f \( -name "Dockerfile*" ! -name "*.md" \) 2>/dev/null)

if [ -z "$DOCKERFILES" ]; then
    echo -e "${YELLOW}⚠️  No Dockerfiles found${NC}"
    WARNINGS=$((WARNINGS + 1))
else
    echo "Found Dockerfiles:"
    echo "$DOCKERFILES" | while read -r dockerfile; do
        echo "  📄 $dockerfile"
    done
    echo ""

    if command_exists hadolint; then
        echo "$DOCKERFILES" | while read -r dockerfile; do
            echo -e "${BLUE}Linting: $dockerfile${NC}"
            
            if hadolint "$dockerfile" 2>&1; then
                echo -e "${GREEN}✅ $dockerfile passed Hadolint validation${NC}"
            else
                echo -e "${RED}❌ $dockerfile has issues${NC}"
                ERRORS=$((ERRORS + 1))
            fi
            echo ""
        done
    fi
fi

#
# 3. MULTI-STAGE BUILD CHECK
#
print_section "3️⃣  Checking Multi-Stage Builds"

echo "$DOCKERFILES" | while read -r dockerfile; do
    if [ -z "$dockerfile" ]; then
        continue
    fi
    
    echo -e "${BLUE}Analyzing: $dockerfile${NC}"
    
    FROM_COUNT=$(grep -c "^FROM " "$dockerfile" || echo "0")
    NAMED_STAGES=$(grep "^FROM .* AS " "$dockerfile" | wc -l | tr -d ' ')
    COPY_FROM=$(grep -c "COPY --from=" "$dockerfile" || echo "0")
    
    echo "  Stages: $FROM_COUNT"
    echo "  Named stages: $NAMED_STAGES"
    echo "  Inter-stage copies: $COPY_FROM"
    
    if [ "$FROM_COUNT" -ge 2 ]; then
        echo -e "  ${GREEN}✅ Multi-stage build detected${NC}"
        
        if [ "$COPY_FROM" -eq 0 ]; then
            echo -e "  ${YELLOW}⚠️  No COPY --from found. Artifacts may not be transferred.${NC}"
            WARNINGS=$((WARNINGS + 1))
        fi
    else
        echo -e "  ${YELLOW}⚠️  Single-stage build. Consider multi-stage for optimization.${NC}"
        WARNINGS=$((WARNINGS + 1))
    fi
    
    # Check final stage base image
    FINAL_BASE=$(grep "^FROM " "$dockerfile" | tail -1 | awk '{print $2}')
    echo "  Final base: $FINAL_BASE"
    
    if [[ "$FINAL_BASE" == *":latest" ]]; then
        echo -e "  ${RED}❌ Using :latest tag - pin to specific version!${NC}"
        ERRORS=$((ERRORS + 1))
    fi
    
    echo ""
done

#
# 4. DOCKER COMPOSE VALIDATION
#
print_section "4️⃣  Validating Docker Compose Files"

COMPOSE_FILES=$(find . -maxdepth 3 -type f \( -name "docker-compose*.yml" -o -name "docker-compose*.yaml" -o -name "compose*.yml" -o -name "compose*.yaml" \) 2>/dev/null)

if [ -z "$COMPOSE_FILES" ]; then
    echo -e "${YELLOW}⚠️  No Docker Compose files found${NC}"
    WARNINGS=$((WARNINGS + 1))
else
    echo "Found Compose files:"
    echo "$COMPOSE_FILES" | while read -r composefile; do
        echo "  📄 $composefile"
    done
    echo ""

    echo "$COMPOSE_FILES" | while read -r composefile; do
        echo -e "${BLUE}Validating: $composefile${NC}"
        
        # Check for obsolete version field
        if grep -q "^version:" "$composefile"; then
            echo -e "  ${RED}❌ Found obsolete 'version' field${NC}"
            echo -e "  ${YELLOW}   Remove 'version:' line - it's obsolete in Compose v2.27.0+${NC}"
            ERRORS=$((ERRORS + 1))
        else
            echo -e "  ${GREEN}✅ No obsolete version field${NC}"
        fi
        
        # Check for :latest tags
        LATEST_COUNT=$(grep -c ":latest" "$composefile" || echo "0")
        if [ "$LATEST_COUNT" -gt 0 ]; then
            echo -e "  ${YELLOW}⚠️  Found $LATEST_COUNT :latest tag(s) - pin to specific versions${NC}"
            WARNINGS=$((WARNINGS + 1))
        fi
        
        # Built-in Docker Compose validation
        if docker compose -f "$composefile" config --quiet 2>&1; then
            echo -e "  ${GREEN}✅ Docker Compose syntax valid${NC}"
        else
            echo -e "  ${RED}❌ Docker Compose syntax errors${NC}"
            docker compose -f "$composefile" config 2>&1 | head -20
            ERRORS=$((ERRORS + 1))
        fi
        
        # DCLint validation
        if command_exists dclint; then
            if dclint "$composefile" 2>&1; then
                echo -e "  ${GREEN}✅ DCLint validation passed${NC}"
            else
                echo -e "  ${YELLOW}⚠️  DCLint found style issues${NC}"
                WARNINGS=$((WARNINGS + 1))
            fi
        fi
        
        echo ""
    done
fi

#
# 5. SECURITY CHECKS
#
print_section "5️⃣  Security Checks"

# Check for root user in final stage
echo "Checking for non-root users in Dockerfiles..."
echo "$DOCKERFILES" | while read -r dockerfile; do
    if [ -z "$dockerfile" ]; then
        continue
    fi
    
    USER_FOUND=$(grep -c "^USER " "$dockerfile" || echo "0")
    LAST_USER=$(grep "^USER " "$dockerfile" | tail -1 | awk '{print $2}')
    
    if [ "$USER_FOUND" -eq 0 ]; then
        echo -e "  ${RED}❌ $dockerfile: No USER specified (runs as root!)${NC}"
        ERRORS=$((ERRORS + 1))
    elif [ "$LAST_USER" == "root" ] || [ "$LAST_USER" == "0" ]; then
        echo -e "  ${RED}❌ $dockerfile: Final USER is root${NC}"
        ERRORS=$((ERRORS + 1))
    else
        echo -e "  ${GREEN}✅ $dockerfile: Running as user '$LAST_USER'${NC}"
    fi
done

#
# 6. BEST PRACTICES CHECK
#
print_section "6️⃣  Best Practices Check"

echo "Dockerfile Best Practices:"
echo "$DOCKERFILES" | while read -r dockerfile; do
    if [ -z "$dockerfile" ]; then
        continue
    fi
    
    echo -e "\n${BLUE}$dockerfile:${NC}"
    
    # Check WORKDIR usage
    if grep -q "^WORKDIR" "$dockerfile"; then
        echo -e "  ${GREEN}✅ Uses WORKDIR${NC}"
    else
        echo -e "  ${YELLOW}⚠️  No WORKDIR specified${NC}"
        WARNINGS=$((WARNINGS + 1))
    fi
    
    # Check HEALTHCHECK
    if grep -q "^HEALTHCHECK" "$dockerfile"; then
        echo -e "  ${GREEN}✅ Has HEALTHCHECK${NC}"
    else
        echo -e "  ${YELLOW}⚠️  No HEALTHCHECK defined${NC}"
        WARNINGS=$((WARNINGS + 1))
    fi
    
    # Check for ADD vs COPY
    ADD_COUNT=$(grep -c "^ADD " "$dockerfile" || echo "0")
    if [ "$ADD_COUNT" -gt 0 ]; then
        echo -e "  ${YELLOW}⚠️  Uses ADD ($ADD_COUNT times) - prefer COPY unless extracting archives${NC}"
        WARNINGS=$((WARNINGS + 1))
    fi
    
    # Check for apt-get cleanup
    if grep -q "apt-get install" "$dockerfile"; then
        if grep -q "rm -rf /var/lib/apt/lists" "$dockerfile"; then
            echo -e "  ${GREEN}✅ Cleans apt cache${NC}"
        else
            echo -e "  ${YELLOW}⚠️  apt-get used but cache not cleaned${NC}"
            WARNINGS=$((WARNINGS + 1))
        fi
    fi
done

echo ""
echo "Docker Compose Best Practices:"
echo "$COMPOSE_FILES" | while read -r composefile; do
    if [ -z "$composefile" ]; then
        continue
    fi
    
    echo -e "\n${BLUE}$composefile:${NC}"
    
    # Check for restart policy
    if grep -q "restart:" "$composefile"; then
        echo -e "  ${GREEN}✅ Has restart policy${NC}"
    else
        echo -e "  ${YELLOW}⚠️  No restart policy defined${NC}"
        WARNINGS=$((WARNINGS + 1))
    fi
    
    # Check for networks
    if grep -q "^networks:" "$composefile"; then
        echo -e "  ${GREEN}✅ Defines networks${NC}"
    else
        echo -e "  ${YELLOW}⚠️  No custom networks defined${NC}"
        WARNINGS=$((WARNINGS + 1))
    fi
    
    # Check for named volumes
    if grep -q "^volumes:" "$composefile"; then
        echo -e "  ${GREEN}✅ Defines named volumes${NC}"
    else
        echo -e "  ${YELLOW}⚠️  No named volumes (data may be lost on restart)${NC}"
        WARNINGS=$((WARNINGS + 1))
    fi
done

#
# FINAL REPORT
#
echo ""
echo -e "${BLUE}╔════════════════════════════════════════════════════════════╗${NC}"
echo -e "${BLUE}║                     Validation Summary                     ║${NC}"
echo -e "${BLUE}╚════════════════════════════════════════════════════════════╝${NC}"
echo ""

if [ $ERRORS -eq 0 ] && [ $WARNINGS -eq 0 ]; then
    echo -e "${GREEN}🎉 Perfect! No errors or warnings found!${NC}"
    exit 0
elif [ $ERRORS -eq 0 ]; then
    echo -e "${YELLOW}⚠️  Validation passed with $WARNINGS warning(s)${NC}"
    exit 0
else
    echo -e "${RED}❌ Validation failed with $ERRORS error(s) and $WARNINGS warning(s)${NC}"
    echo ""
    echo "Please fix the errors above before proceeding."
    exit 1
fi
```

### Make Script Executable

```bash
chmod +x validate-docker.sh

# Run validation
./validate-docker.sh
```

---

## CI/CD Integration

### GitHub Actions

Create `.github/workflows/docker-validation.yml`:

```yaml
name: Docker Validation

on:
  push:
    branches: [ main, develop ]
  pull_request:
    branches: [ main ]
  workflow_dispatch:

jobs:
  validate-docker:
    runs-on: ubuntu-latest
    
    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Set up Hadolint
        uses: hadolint/hadolint-action@v3.1.0
        with:
          dockerfile: ./Dockerfile
          failure-threshold: error
          
      - name: Lint all Dockerfiles
        run: |
          find . -name "Dockerfile*" -type f | while read dockerfile; do
            echo "Linting $dockerfile"
            hadolint "$dockerfile" || exit 1
          done

      - name: Validate Docker Compose files
        run: |
          find . -name "docker-compose*.yml" -o -name "compose*.yml" | while read composefile; do
            echo "Validating $composefile"
            docker compose -f "$composefile" config --quiet || exit 1
          done

      - name: Install DCLint
        run: npm install -g docker-compose-linter

      - name: Run DCLint
        run: |
          find . -name "docker-compose*.yml" -o -name "compose*.yml" | while read composefile; do
            echo "Linting $composefile"
            dclint "$composefile" || exit 1
          done

      - name: Check for multi-stage builds
        run: |
          ./scripts/validate-multistage.sh Dockerfile

      - name: Security scan with Trivy
        uses: aquasecurity/trivy-action@master
        with:
          scan-type: 'config'
          scan-ref: '.'
          format: 'sarif'
          output: 'trivy-results.sarif'

      - name: Upload Trivy results to GitHub Security
        uses: github/codeql-action/upload-sarif@v2
        if: always()
        with:
          sarif_file: 'trivy-results.sarif'
```

### GitLab CI

Create `.gitlab-ci.yml`:

```yaml
stages:
  - validate

variables:
  HADOLINT_VERSION: "v2.12.0"

docker-validation:
  stage: validate
  image: hadolint/hadolint:latest-debian
  script:
    # Validate all Dockerfiles
    - find . -name "Dockerfile*" -type f -exec hadolint {} \;
    
    # Check for multi-stage builds
    - |
      for dockerfile in $(find . -name "Dockerfile*" -type f); do
        STAGES=$(grep -c "^FROM " "$dockerfile")
        if [ "$STAGES" -lt 2 ]; then
          echo "Warning: $dockerfile is not multi-stage"
        fi
      done
  only:
    - merge_requests
    - main
    - develop

compose-validation:
  stage: validate
  image: docker:latest
  services:
    - docker:dind
  before_script:
    - apk add --no-cache nodejs npm
    - npm install -g docker-compose-linter
  script:
    # Validate Docker Compose syntax
    - |
      for composefile in $(find . -name "docker-compose*.yml" -o -name "compose*.yml"); do
        echo "Validating $composefile"
        docker compose -f "$composefile" config --quiet
        dclint "$composefile"
      done
    
    # Check for obsolete version field
    - |
      if grep -r "^version:" . --include="*compose*.yml"; then
        echo "ERROR: Found obsolete 'version' field in compose file(s)"
        exit 1
      fi
  only:
    - merge_requests
    - main
    - develop
```

### Pre-commit Hook

Create `.git/hooks/pre-commit`:

```bash
#!/bin/bash
# Pre-commit hook for Docker validation

echo "🔍 Running Docker validation..."

# Find all staged Dockerfiles
STAGED_DOCKERFILES=$(git diff --cached --name-only --diff-filter=ACM | grep -E 'Dockerfile.*$')

if [ -n "$STAGED_DOCKERFILES" ]; then
    echo "📄 Validating Dockerfiles..."
    
    for dockerfile in $STAGED_DOCKERFILES; do
        if command -v hadolint >/dev/null 2>&1; then
            echo "  Linting $dockerfile"
            hadolint "$dockerfile" || exit 1
        fi
    done
fi

# Find all staged Compose files
STAGED_COMPOSE=$(git diff --cached --name-only --diff-filter=ACM | grep -E '(docker-)?compose.*\.ya?ml$')

if [ -n "$STAGED_COMPOSE" ]; then
    echo "📄 Validating Docker Compose files..."
    
    for composefile in $STAGED_COMPOSE; do
        # Check for obsolete version field
        if grep -q "^version:" "$composefile"; then
            echo "❌ ERROR: $composefile contains obsolete 'version' field"
            exit 1
        fi
        
        # Validate syntax
        docker compose -f "$composefile" config --quiet || exit 1
    done
fi

echo "✅ Docker validation passed!"
exit 0
```

Make it executable:

```bash
chmod +x .git/hooks/pre-commit
```

---

## Common Issues & Solutions

### Issue 1: Hadolint DL3008 - Pin versions in apt-get

**Problem:**
```dockerfile
RUN apt-get update && apt-get install -y curl
```

**Solution:**
```dockerfile
RUN apt-get update && \
    apt-get install -y \
        curl=7.68.0-1ubuntu2.14 && \
    rm -rf /var/lib/apt/lists/*
```

### Issue 2: Docker Compose "version is obsolete"

**Problem:**
```yaml
version: '3.8'  # ❌ Obsolete
services:
  web:
    image: nginx
```

**Solution:**
```yaml
# Just remove the version field entirely!
services:
  web:
    image: nginx:1.24-alpine
```

### Issue 3: Non-Multi-Stage Build

**Problem:**
```dockerfile
FROM node:18
COPY . .
RUN npm install
RUN npm run build
CMD ["npm", "start"]
```

**Solution:**
```dockerfile
# Build stage
FROM node:18 AS builder
COPY . .
RUN npm ci && npm run build

# Runtime stage
FROM node:18-alpine
COPY --from=builder /app/dist ./dist
CMD ["node", "dist/index.js"]
```

### Issue 4: Running as Root

**Problem:**
```dockerfile
FROM alpine
COPY app /app
CMD ["/app/server"]
```

**Solution:**
```dockerfile
FROM alpine
RUN addgroup -g 1001 appuser && \
    adduser -D -u 1001 -G appuser appuser
COPY --chown=appuser:appuser app /app
USER appuser
CMD ["/app/server"]
```

### Issue 5: Using :latest tag

**Problem:**
```dockerfile
FROM python:latest
```

**Solution:**
```dockerfile
FROM python:3.11-slim-bookworm
```

---

## Best Practices Checklist

### Dockerfile Checklist

- [ ] Uses multi-stage builds (when applicable)
- [ ] All stages are named with `AS`
- [ ] Final stage uses minimal base image (alpine/slim)
- [ ] No `:latest` tags
- [ ] Specifies `USER` (non-root)
- [ ] Uses `WORKDIR` instead of `cd`
- [ ] Includes `HEALTHCHECK`
- [ ] Uses `COPY` instead of `ADD` (unless extracting)
- [ ] Cleans package manager cache (apt/apk/yum)
- [ ] Combines RUN commands to reduce layers
- [ ] Uses `.dockerignore` file
- [ ] Pins dependency versions
- [ ] Labels included (maintainer, version, etc.)
- [ ] Passes Hadolint validation
- [ ] Build tools excluded from final image

### Docker Compose Checklist

- [ ] No `version` field (obsolete)
- [ ] All images use specific tags (no `:latest`)
- [ ] Restart policies defined
- [ ] Named volumes for persistence
- [ ] Custom networks defined
- [ ] Health checks configured
- [ ] Resource limits set (optional but recommended)
- [ ] Environment variables properly managed
- [ ] Secrets not hardcoded (use `.env` files)
- [ ] Service dependencies specified (`depends_on`)
- [ ] Passes `docker compose config` validation
- [ ] Passes DCLint validation

### Security Checklist

- [ ] Runs as non-root user
- [ ] Minimal base images used
- [ ] No secrets in Dockerfile or Compose files
- [ ] Regular security scans (Trivy/Snyk)
- [ ] Network policies configured
- [ ] Read-only root filesystem (where possible)
- [ ] Capabilities dropped where not needed
- [ ] No unnecessary ports exposed

### CI/CD Checklist

- [ ] Automated validation in pipeline
- [ ] Pre-commit hooks configured
- [ ] Validation fails on errors
- [ ] Reports generated and archived
- [ ] Notifications on failures
- [ ] Regular dependency updates
- [ ] Image scanning integrated

---

## Quick Reference Commands

```bash
# Validate Dockerfile
hadolint Dockerfile

# Validate Dockerfile with Docker
docker run --rm -i hadolint/hadolint < Dockerfile

# Validate Docker Compose
docker compose config --quiet

# Lint Docker Compose
dclint docker-compose.yml

# Fix Docker Compose issues
dclint --fix docker-compose.yml

# Check Docker info
docker system info

# View Docker storage driver
docker info | grep "Storage Driver"

# List all containers
docker ps -a

# Check multi-stage build
grep -c "^FROM " Dockerfile

# Find all Dockerfiles
find . -name "Dockerfile*" -type f

# Find all Compose files
find . -name "*compose*.yml" -type f

# Run full validation
./validate-docker.sh
```

---

## Additional Resources

### Documentation
- [Docker Best Practices](https://docs.docker.com/develop/dev-best-practices/)
- [Dockerfile Reference](https://docs.docker.com/engine/reference/builder/)
- [Compose Specification](https://docs.docker.com/compose/compose-file/)
- [Hadolint Rules](https://github.com/hadolint/hadolint#rules)

### Tools
- [Hadolint](https://github.com/hadolint/hadolint) - Dockerfile linter
- [DCLint](https://github.com/zavoloklom/docker-compose-linter) - Compose linter
- [Trivy](https://github.com/aquasecurity/trivy) - Security scanner
- [Dive](https://github.com/wagoodman/dive) - Image layer analyzer
- [Docker Bench Security](https://github.com/docker/docker-bench-security) - Security audit

### Learning Resources
- [Play with Docker](https://labs.play-with-docker.com/)
- [Docker Curriculum](https://docker-curriculum.com/)
- [Dockerfile Best Practices Guide](https://docs.docker.com/develop/develop-images/dockerfile_best-practices/)

---

## Troubleshooting

### Hadolint Not Finding .hadolint.yaml

```bash
# Check current directory
ls -la .hadolint.yaml

# Run with explicit config
hadolint --config .hadolint.yaml Dockerfile

# Check if hadolint sees the config
hadolint --version --config .hadolint.yaml
```

### Docker Compose Config Fails

```bash
# Get detailed error
docker compose config

# Check syntax
docker compose -f docker-compose.yml config --quiet 2>&1

# Validate specific services
docker compose config --services
```

### CI/CD Pipeline Fails

```bash
# Run locally first
./validate-docker.sh

# Check individual tools
hadolint --version
docker compose version
dclint --version

# Test in Docker container
docker run --rm -v $(pwd):/app -w /app hadolint/hadolint Dockerfile
```

---

## Version History

| Version | Date | Changes |
|---------|------|---------|
| 1.0.0 | 2025-10-18 | Initial comprehensive guide |

---

## Support & Contributions

For issues, suggestions, or contributions, please refer to your project's repository or documentation.

**Remember:** Validation is only the first step. Regular reviews, security scans, and keeping dependencies updated are essential for maintaining secure and efficient Docker configurations.

---

**Happy Dockering! 🐳**
