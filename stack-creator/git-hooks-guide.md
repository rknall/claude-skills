# Git Hooks and Validation Scripts Guide

This guide provides comprehensive examples and guidelines for git hooks and validation scripts used in GitLab Stack projects.

## Overview

Git hooks are scripts that run automatically at specific points in the git workflow. For GitLab Stack projects, we use hooks to ensure validation before commits.

## Directory Structure

```
project-name/
├── .git/
│   └── hooks/
│       └── pre-commit         # Installed hook
└── scripts/
    ├── pre-commit              # Source hook script
    ├── validate-stack.sh       # Full validation
    └── setup-hooks.sh          # Hook installer
```

## Core Scripts

### 1. scripts/pre-commit

The pre-commit hook runs before each commit to validate the stack.

**Location**: `scripts/pre-commit`
**Installed to**: `.git/hooks/pre-commit`
**When it runs**: Before `git commit`

```bash
#!/usr/bin/env bash
#
# Pre-commit hook for GitLab Stack validation
# Prevents commits that violate stack patterns
#

set -euo pipefail

# Colors for output
RED='\033[0;31m'
GREEN='\033[0;32m'
YELLOW='\033[1;33m'
NC='\033[0m' # No Color

echo "========================================="
echo "Pre-commit validation"
echo "========================================="
echo

ERRORS=0

# Check 1: Secrets in staged files
echo "1. Checking for secrets in staged files..."
if git diff --cached --name-only | grep -qE "secrets/.*[^.gitkeep]|\.env$"; then
    echo -e "${RED}✗ ERROR: Attempting to commit secrets or .env file!${NC}"
    echo "  Secrets should NEVER be committed to git."
    echo "  Files detected:"
    git diff --cached --name-only | grep -E "secrets/.*[^.gitkeep]|\.env$" | sed 's/^/    /'
    ((ERRORS++))
else
    echo -e "${GREEN}✓ No secrets in staged files${NC}"
fi
echo

# Check 2: Root-owned files
echo "2. Checking for root-owned files..."
if find . -user root -not -path "./.git/*" 2>/dev/null | grep -q .; then
    echo -e "${RED}✗ ERROR: Root-owned files detected!${NC}"
    echo "  All files should be owned by the user running Docker."
    echo "  Files detected:"
    find . -user root -not -path "./.git/*" 2>/dev/null | sed 's/^/    /'
    echo
    echo "  Fix with: sudo chown -R \$USER:\$USER ."
    ((ERRORS++))
else
    echo -e "${GREEN}✓ No root-owned files${NC}"
fi
echo

# Check 3: Secrets in file content (basic check)
echo "3. Checking for hardcoded secrets in code..."
SUSPICIOUS_PATTERNS=(
    "password\s*=\s*['\"][^'\"]+['\"]"
    "api[_-]?key\s*=\s*['\"][^'\"]+['\"]"
    "secret\s*=\s*['\"][^'\"]+['\"]"
    "token\s*=\s*['\"][^'\"]+['\"]"
)

FOUND_SECRETS=false
for pattern in "${SUSPICIOUS_PATTERNS[@]}"; do
    if git diff --cached | grep -iE "$pattern" | grep -v "\.example" | grep -q .; then
        if [ "$FOUND_SECRETS" = false ]; then
            echo -e "${YELLOW}⚠ WARNING: Possible hardcoded secrets detected:${NC}"
            FOUND_SECRETS=true
        fi
        git diff --cached | grep -iE "$pattern" | grep -v "\.example" | sed 's/^/    /'
    fi
done

if [ "$FOUND_SECRETS" = true ]; then
    echo
    echo "  Review these carefully. Use environment variables or Docker secrets instead."
    echo "  If these are false positives, you can proceed."
else
    echo -e "${GREEN}✓ No obvious hardcoded secrets${NC}"
fi
echo

# Check 4: Full stack validation (if available)
if [ -x "./scripts/validate-stack.sh" ]; then
    echo "4. Running full stack validation..."
    if ! ./scripts/validate-stack.sh; then
        echo -e "${RED}✗ ERROR: Stack validation failed!${NC}"
        echo "  Fix all issues before committing."
        echo "  Or use 'git commit --no-verify' to skip (NOT recommended)."
        ((ERRORS++))
    else
        echo -e "${GREEN}✓ Stack validation passed${NC}"
    fi
else
    echo -e "${YELLOW}⚠ Skipping stack validation (./scripts/validate-stack.sh not found)${NC}"
fi
echo

# Final result
echo "========================================="
if [ $ERRORS -gt 0 ]; then
    echo -e "${RED}Pre-commit validation FAILED with $ERRORS error(s)${NC}"
    echo "========================================="
    echo
    echo "To skip this validation (NOT recommended):"
    echo "  git commit --no-verify"
    exit 1
fi

echo -e "${GREEN}Pre-commit validation PASSED!${NC}"
echo "========================================="
exit 0
```

**Usage**:
```bash
# Automatic - runs on every commit
git commit -m "message"

# Skip validation (emergency only)
git commit --no-verify -m "message"
```

---

### 2. scripts/validate-stack.sh

Comprehensive validation script that runs all validators.

**Location**: `scripts/validate-stack.sh`
**When to run**: Before deployment, in CI/CD, or manually

```bash
#!/usr/bin/env bash
#
# Full GitLab Stack validation
# Runs all validators to ensure stack compliance
#

set -euo pipefail

# Colors for output
RED='\033[0;31m'
GREEN='\033[0;32m'
YELLOW='\033[1;33m'
BLUE='\033[0;34m'
NC='\033[0m' # No Color

echo
echo -e "${BLUE}========================================"
echo "GitLab Stack Validation"
echo "========================================${NC}"
echo

ERRORS=0
WARNINGS=0

# Check if we're in a stack directory
if [ ! -f "docker-compose.yml" ]; then
    echo -e "${RED}✗ ERROR: docker-compose.yml not found!${NC}"
    echo "  Are you in a stack directory?"
    exit 1
fi

# Validation 1: Stack Validator
echo -e "${BLUE}[1/4] Running stack-validator...${NC}"
if command -v claude-code >/dev/null 2>&1; then
    if claude-code run stack-validator 2>&1 | tee /tmp/stack-validator.log; then
        echo -e "${GREEN}✓ Stack validation passed${NC}"
    else
        echo -e "${RED}✗ Stack validation failed${NC}"
        echo "  Review output above for details"
        ((ERRORS++))
    fi
else
    echo -e "${YELLOW}⚠ Claude Code not available, skipping stack-validator${NC}"
    ((WARNINGS++))
fi
echo

# Validation 2: Secrets Manager
echo -e "${BLUE}[2/4] Running secrets-manager validation...${NC}"
if command -v claude-code >/dev/null 2>&1; then
    if claude-code run secrets-manager --validate 2>&1 | tee /tmp/secrets-manager.log; then
        echo -e "${GREEN}✓ Secrets validation passed${NC}"
    else
        echo -e "${RED}✗ Secrets validation failed${NC}"
        echo "  Review output above for details"
        ((ERRORS++))
    fi
else
    echo -e "${YELLOW}⚠ Claude Code not available, running basic secrets check${NC}"

    # Basic secrets check without Claude Code
    if find secrets/ -type f ! -name ".gitkeep" 2>/dev/null | grep -q .; then
        if grep -r "DOCKER_SECRET" docker-compose.yml >/dev/null 2>&1; then
            echo -e "${GREEN}✓ Basic secrets check passed${NC}"
        else
            echo -e "${YELLOW}⚠ Secrets files found but not referenced in docker-compose.yml${NC}"
            ((WARNINGS++))
        fi
    else
        echo -e "${GREEN}✓ No secrets configured${NC}"
    fi
fi
echo

# Validation 3: Docker Validator
echo -e "${BLUE}[3/4] Running docker-validation...${NC}"
if command -v claude-code >/dev/null 2>&1; then
    if claude-code run docker-validation 2>&1 | tee /tmp/docker-validation.log; then
        echo -e "${GREEN}✓ Docker validation passed${NC}"
    else
        echo -e "${RED}✗ Docker validation failed${NC}"
        echo "  Review output above for details"
        ((ERRORS++))
    fi
else
    echo -e "${YELLOW}⚠ Claude Code not available, running basic Docker checks${NC}"

    # Basic docker-compose syntax check
    if docker compose config >/dev/null 2>&1; then
        echo -e "${GREEN}✓ docker-compose.yml syntax valid${NC}"
    else
        echo -e "${RED}✗ docker-compose.yml syntax invalid${NC}"
        ((ERRORS++))
    fi
fi
echo

# Validation 4: File Ownership
echo -e "${BLUE}[4/4] Checking file ownership...${NC}"
if find . -user root -not -path "./.git/*" 2>/dev/null | grep -q .; then
    echo -e "${RED}✗ Root-owned files detected:${NC}"
    find . -user root -not -path "./.git/*" 2>/dev/null | sed 's/^/    /'
    echo
    echo "  Fix with: sudo chown -R \$USER:\$USER ."
    ((ERRORS++))
else
    echo -e "${GREEN}✓ No root-owned files${NC}"
fi
echo

# Additional checks
echo -e "${BLUE}Additional checks:${NC}"

# Check .env vs .env.example sync
if [ -f ".env.example" ]; then
    ENV_KEYS=$(grep -v '^#' .env.example 2>/dev/null | grep '=' | cut -d= -f1 | sort)
    if [ -f ".env" ]; then
        ACTUAL_KEYS=$(grep -v '^#' .env 2>/dev/null | grep '=' | cut -d= -f1 | sort)
        if [ "$ENV_KEYS" != "$ACTUAL_KEYS" ]; then
            echo -e "${YELLOW}⚠ .env and .env.example keys don't match${NC}"
            ((WARNINGS++))
        else
            echo -e "${GREEN}✓ .env synced with .env.example${NC}"
        fi
    else
        echo -e "${YELLOW}⚠ .env file not found (expected from .env.example)${NC}"
        ((WARNINGS++))
    fi
fi

# Check git setup
if [ -d ".git" ]; then
    # Check default branch
    DEFAULT_BRANCH=$(git config init.defaultBranch 2>/dev/null || echo "")
    if [ "$DEFAULT_BRANCH" = "main" ]; then
        echo -e "${GREEN}✓ Git default branch: main${NC}"
    else
        echo -e "${YELLOW}⚠ Git default branch not set to 'main'${NC}"
        ((WARNINGS++))
    fi

    # Check merge strategy
    MERGE_FF=$(git config merge.ff 2>/dev/null || echo "")
    if [ "$MERGE_FF" = "only" ]; then
        echo -e "${GREEN}✓ Git merge strategy: ff-only${NC}"
    else
        echo -e "${YELLOW}⚠ Git merge strategy not set to 'ff-only'${NC}"
        ((WARNINGS++))
    fi
else
    echo -e "${YELLOW}⚠ Not a git repository${NC}"
    ((WARNINGS++))
fi

echo

# Final report
echo -e "${BLUE}========================================"
echo "Validation Summary"
echo "========================================${NC}"

if [ $ERRORS -gt 0 ]; then
    echo -e "${RED}FAILED: $ERRORS error(s) found${NC}"
    if [ $WARNINGS -gt 0 ]; then
        echo -e "${YELLOW}$WARNINGS warning(s) found${NC}"
    fi
    echo "========================================${NC}"
    exit 1
elif [ $WARNINGS -gt 0 ]; then
    echo -e "${YELLOW}PASSED with $WARNINGS warning(s)${NC}"
    echo "========================================${NC}"
    exit 0
else
    echo -e "${GREEN}ALL VALIDATIONS PASSED!${NC}"
    echo "========================================${NC}"
    exit 0
fi
```

**Usage**:
```bash
# Run full validation
./scripts/validate-stack.sh

# In CI/CD
./scripts/validate-stack.sh || exit 1
```

---

### 3. scripts/setup-hooks.sh

Script to install git hooks from scripts/ to .git/hooks/

**Location**: `scripts/setup-hooks.sh`
**When to run**: After cloning, during stack creation

```bash
#!/usr/bin/env bash
#
# Install git hooks from scripts/ to .git/hooks/
#

set -euo pipefail

# Colors
GREEN='\033[0;32m'
RED='\033[0;31m'
YELLOW='\033[1;33m'
NC='\033[0m'

echo "Installing git hooks..."
echo

# Check if .git exists
if [ ! -d ".git" ]; then
    echo -e "${RED}✗ ERROR: Not a git repository!${NC}"
    echo "  Initialize git first: git init"
    exit 1
fi

# Ensure hooks directory exists
mkdir -p .git/hooks

INSTALLED=0
FAILED=0

# Install pre-commit hook
if [ -f "scripts/pre-commit" ]; then
    if cp scripts/pre-commit .git/hooks/pre-commit; then
        chmod +x .git/hooks/pre-commit
        echo -e "${GREEN}✓ Installed pre-commit hook${NC}"
        ((INSTALLED++))
    else
        echo -e "${RED}✗ Failed to install pre-commit hook${NC}"
        ((FAILED++))
    fi
else
    echo -e "${YELLOW}⚠ scripts/pre-commit not found, skipping${NC}"
fi

# Install pre-push hook (if exists)
if [ -f "scripts/pre-push" ]; then
    if cp scripts/pre-push .git/hooks/pre-push; then
        chmod +x .git/hooks/pre-push
        echo -e "${GREEN}✓ Installed pre-push hook${NC}"
        ((INSTALLED++))
    else
        echo -e "${RED}✗ Failed to install pre-push hook${NC}"
        ((FAILED++))
    fi
fi

echo
echo "========================================="
if [ $FAILED -gt 0 ]; then
    echo -e "${RED}Installation completed with $FAILED error(s)${NC}"
    echo "========================================="
    exit 1
elif [ $INSTALLED -eq 0 ]; then
    echo -e "${YELLOW}No hooks installed${NC}"
    echo "========================================="
    exit 0
else
    echo -e "${GREEN}Successfully installed $INSTALLED hook(s)!${NC}"
    echo "========================================="
    echo
    echo "Hooks will now run automatically:"
    echo "  - pre-commit: Before each commit"
    echo
    echo "To skip hooks (emergency only):"
    echo "  git commit --no-verify"
fi
```

**Usage**:
```bash
# Install hooks
./scripts/setup-hooks.sh

# Verify installation
ls -la .git/hooks/
```

---

## Optional Hooks

### scripts/pre-push (Optional)

Runs before `git push` to ensure remote-ready state.

```bash
#!/usr/bin/env bash
#
# Pre-push hook for GitLab Stack
# Runs before pushing to remote
#

set -euo pipefail

# Colors
RED='\033[0;31m'
GREEN='\033[0;32m'
YELLOW='\033[1;33m'
NC='\033[0m'

echo "========================================="
echo "Pre-push validation"
echo "========================================="
echo

ERRORS=0

# Run full validation before push
echo "Running full stack validation..."
if [ -x "./scripts/validate-stack.sh" ]; then
    if ! ./scripts/validate-stack.sh; then
        echo -e "${RED}✗ Stack validation failed!${NC}"
        echo "  Fix issues before pushing."
        ((ERRORS++))
    fi
else
    echo -e "${YELLOW}⚠ ./scripts/validate-stack.sh not found${NC}"
fi

# Check for uncommitted changes
if ! git diff-index --quiet HEAD --; then
    echo -e "${YELLOW}⚠ WARNING: You have uncommitted changes${NC}"
    echo "  Consider committing them before pushing."
fi

echo
if [ $ERRORS -gt 0 ]; then
    echo -e "${RED}Pre-push validation FAILED${NC}"
    echo "To skip: git push --no-verify"
    exit 1
fi

echo -e "${GREEN}Pre-push validation PASSED${NC}"
exit 0
```

---

## CI/CD Integration

### GitLab CI (.gitlab-ci.yml)

```yaml
stages:
  - validate
  - build
  - deploy

validate:
  stage: validate
  image: docker:latest
  services:
    - docker:dind
  before_script:
    - apk add --no-cache bash findutils
  script:
    - chmod +x ./scripts/validate-stack.sh
    - ./scripts/validate-stack.sh
  only:
    - merge_requests
    - main
```

### GitHub Actions (.github/workflows/validate.yml)

```yaml
name: Stack Validation

on:
  push:
    branches: [ main ]
  pull_request:
    branches: [ main ]

jobs:
  validate:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3

      - name: Run stack validation
        run: |
          chmod +x ./scripts/validate-stack.sh
          ./scripts/validate-stack.sh
```

---

## Best Practices

### 1. Always Make Scripts Executable

```bash
chmod +x scripts/*.sh
chmod +x scripts/pre-commit
```

### 2. Test Hooks Before Committing

```bash
# Test pre-commit manually
./scripts/pre-commit

# Test validation
./scripts/validate-stack.sh
```

### 3. Document Hook Behavior

Include in README.md:
```markdown
## Git Hooks

This project uses git hooks for validation:
- **pre-commit**: Validates before each commit
- To skip: `git commit --no-verify` (emergency only)
```

### 4. Provide Skip Option

Always allow users to skip in emergencies:
```bash
git commit --no-verify -m "emergency fix"
```

### 5. Keep Hooks Fast

- Pre-commit should run in < 10 seconds
- Use quick checks when possible
- Defer expensive checks to CI/CD

### 6. Clear Error Messages

```bash
echo -e "${RED}✗ ERROR: Clear description${NC}"
echo "  Explanation of what went wrong"
echo "  How to fix it"
```

### 7. Exit Codes

```bash
# Success
exit 0

# Failure
exit 1

# Always use set -e to catch errors
set -euo pipefail
```

---

## Troubleshooting

### Hook Not Running

```bash
# Check if hook is installed
ls -la .git/hooks/pre-commit

# Check if executable
chmod +x .git/hooks/pre-commit

# Reinstall hooks
./scripts/setup-hooks.sh
```

### Hook Fails Unexpectedly

```bash
# Run hook manually to see output
./scripts/pre-commit

# Check validation separately
./scripts/validate-stack.sh

# Debug with set -x
bash -x scripts/pre-commit
```

### Skip Hook Temporarily

```bash
# Skip pre-commit
git commit --no-verify -m "message"

# Skip pre-push
git push --no-verify
```

### Permission Denied

```bash
# Make script executable
chmod +x scripts/pre-commit
chmod +x scripts/validate-stack.sh

# Reinstall hooks
./scripts/setup-hooks.sh
```

---

## Customization

### Adding Custom Checks

Edit `scripts/pre-commit`:

```bash
# Add custom check
echo "5. Running custom validation..."
if ! ./scripts/my-custom-check.sh; then
    echo -e "${RED}✗ Custom validation failed${NC}"
    ((ERRORS++))
else
    echo -e "${GREEN}✓ Custom validation passed${NC}"
fi
echo
```

### Adjusting Validation Strictness

**Strict Mode** (recommended for production):
```bash
# Fail on any error
set -euo pipefail
```

**Lenient Mode** (development only):
```bash
# Continue on errors, just report
set -uo pipefail
```

### Environment-Specific Hooks

```bash
# Check environment
if [ "${ENV:-}" = "production" ]; then
    # Strict validation for production
    ./scripts/validate-stack.sh
else
    # Lenient for development
    echo "Development environment, skipping some checks"
fi
```

---

## Summary

Git hooks ensure:
- ✅ No secrets committed
- ✅ No root-owned files
- ✅ Full stack validation before commit
- ✅ Consistent code quality
- ✅ Automated validation in workflow

All scripts are:
- Executable (`chmod +x`)
- Well-documented
- Provide clear error messages
- Support emergency skip (`--no-verify`)
- Integrate with CI/CD
