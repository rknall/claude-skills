---
name: "GitLab Stack Creator"
description: "Create new GitLab stack projects from templates with proper directory structure, git configuration, validation hooks, and documentation. Use when initializing new Docker stack projects that follow GitLab stack patterns."
---

# GitLab Stack Creator

Expert assistance for creating new GitLab stack projects with proper structure, git configuration, validation scripts, and comprehensive documentation.

## When to Use This Skill

This skill should be triggered when:
- Creating a new GitLab stack project
- Initializing a Docker stack with proper structure
- Setting up a project with ./config, ./secrets, ./_temporary directories
- Configuring git repository for stack projects
- Setting up validation hooks and scripts
- Bootstrapping a stack project with best practices

## Core Principles

This skill follows the GitLab Stack Management patterns:

1. **Everything configured through docker-compose.yml and ./config**
2. **Secrets stored in ./secrets and Docker secrets**
3. **docker-entrypoint.sh scripts only when containers don't support native secrets**
4. **All container files owned by the user running docker (no root-owned files)**
5. **./_temporary directory for transient setup files (cleaned up after use)**
6. **Complete validation before considering stack creation complete**

## Stack Creation is Complete When

A stack creation is considered COMPLETE only when:

1. ✅ **stack-validator** skill reports NO issues
2. ✅ **secrets-manager** skill is satisfied with NO open issues
3. ✅ **docker-validation** skill is satisfied with NO issues
4. ✅ All validation scripts execute successfully
5. ✅ Git repository is properly initialized and configured
6. ✅ Documentation is complete in ./docs

**IMPORTANT**: NEVER use workarounds. If the direct approach is not working, STOP and ask the user what to do instead.

## Stack Creation Workflow

### Phase 1: Project Initialization

#### 1.1 Gather Requirements

Ask the user:
- Project name
- Primary services needed (nginx, PostgreSQL, Redis, etc.)
- Remote git repository URL (if any)
- Environment (development, staging, production)
- Special requirements

**NEVER assume** - always ask if information is missing.

#### 1.2 Check Existing Setup

Before creating anything:
- Check if directory already exists
- Check if git repository exists
- Ask user how to proceed if conflicts exist
- NEVER overwrite without explicit permission

### Phase 2: Directory Structure Creation

Create the standard directory structure:

```
project-name/
├── .git/                       # Git repository
│   ├── hooks/                  # Git hooks (validation scripts)
│   └── config                  # Git configuration
├── config/                     # Service configurations
│   ├── nginx/                  # Nginx configs (if needed)
│   ├── postgres/               # PostgreSQL configs (if needed)
│   └── redis/                  # Redis configs (if needed)
├── secrets/                    # Docker secrets files
│   └── .gitkeep               # Keep directory in git
├── _temporary/                 # Temporary files (gitignored)
├── scripts/                    # Validation and utility scripts
│   ├── pre-commit              # Pre-commit validation hook
│   ├── validate-stack.sh       # Full stack validation
│   └── setup-hooks.sh          # Hook installation script
├── docs/                       # Project documentation
│   ├── decisions/              # Architecture decision records
│   ├── setup.md                # Setup instructions
│   └── services.md             # Service documentation
├── docker-compose.yml          # Main compose file
├── .env.example                # Environment template
├── .gitignore                  # Git exclusions
├── .dockerignore               # Docker exclusions
├── CLAUDE.md                   # Claude Code instructions
└── README.md                   # Project overview
```

**Actions**:
1. Create directories: `mkdir -p config secrets _temporary scripts docs/decisions`
2. Create placeholder files: `touch secrets/.gitkeep`
3. Set proper permissions: `chmod 700 secrets`

### Phase 3: Git Repository Setup

#### 3.1 Initialize or Connect Repository

**If NO git repository exists**:
- Strongly suggest creating a local repository
- Ask if remote repository should be added
- Initialize with `git init`
- Set main as default branch: `git config init.defaultBranch main`

**If remote repository URL provided**:
- Clone repository: `git clone <url> <project-name>`
- Or add remote: `git remote add origin <url>`
- Verify remote connection: `git remote -v`

**If setup fails**: STOP and ask user for guidance. NEVER attempt workarounds.

#### 3.2 Configure Git

Set required git configuration:
```bash
# Set main as branch name
git config init.defaultBranch main

# Set ff-only merge strategy
git config pull.ff only
git config merge.ff only

# Ensure no rebase by default
git config pull.rebase false

# Set user info if not set globally
git config user.name "User Name"
git config user.email "user@example.com"
```

**If configuration fails**: Ask user to set configuration manually or provide correct values.

#### 3.3 Create .gitignore

Generate comprehensive .gitignore:
```gitignore
# Secrets - NEVER commit
secrets/*
!secrets/.gitkeep
*.key
*.pem
*.crt
*.p12
*.pfx

# Environment files
.env
.env.local
.env.*.local

# Temporary files
_temporary/
*.tmp
*.temp
.cache/

# Docker
.docker/

# IDE
.vscode/
.idea/
*.swp
*.swo
*~

# OS
.DS_Store
Thumbs.db
*.log

# Backup files
*.bak
*.backup
```

#### 3.4 Create .dockerignore

Generate .dockerignore:
```dockerignore
.git
.gitignore
README.md
docs/
_temporary/
*.md
.env
.env.example
secrets/
.vscode/
.idea/
```

### Phase 4: Validation Scripts Setup

#### 4.1 Create scripts/validate-stack.sh

Generate comprehensive validation script that:
- Calls stack-validator skill
- Calls secrets-manager skill validation
- Calls docker-validation skill
- Reports all issues
- Returns exit code 0 only if all pass

**Template**:
```bash
#!/usr/bin/env bash
set -euo pipefail

echo "=== GitLab Stack Validation ==="
echo

# Colors for output
RED='\033[0;31m'
GREEN='\033[0;32m'
YELLOW='\033[1;33m'
NC='\033[0m' # No Color

ERRORS=0

echo "1. Validating stack structure..."
if ! claude-code-cli run stack-validator; then
    echo -e "${RED}✗ Stack validation failed${NC}"
    ((ERRORS++))
else
    echo -e "${GREEN}✓ Stack validation passed${NC}"
fi
echo

echo "2. Validating secrets configuration..."
if ! claude-code-cli run secrets-manager --validate; then
    echo -e "${RED}✗ Secrets validation failed${NC}"
    ((ERRORS++))
else
    echo -e "${GREEN}✓ Secrets validation passed${NC}"
fi
echo

echo "3. Validating Docker configuration..."
if ! claude-code-cli run docker-validation; then
    echo -e "${RED}✗ Docker validation failed${NC}"
    ((ERRORS++))
else
    echo -e "${GREEN}✓ Docker validation passed${NC}"
fi
echo

if [ $ERRORS -gt 0 ]; then
    echo -e "${RED}Validation failed with $ERRORS error(s)${NC}"
    exit 1
fi

echo -e "${GREEN}All validations passed!${NC}"
exit 0
```

Make executable: `chmod +x scripts/validate-stack.sh`

#### 4.2 Create scripts/pre-commit

Generate pre-commit hook:
```bash
#!/usr/bin/env bash
set -euo pipefail

echo "Running pre-commit validation..."

# Check for secrets in staged files
if git diff --cached --name-only | grep -qE "secrets/.*[^.gitkeep]|\.env$"; then
    echo "ERROR: Attempting to commit secrets or .env file!"
    echo "Secrets should never be committed to git."
    exit 1
fi

# Check for root-owned files
if find . -user root -not -path "./.git/*" 2>/dev/null | grep -q .; then
    echo "ERROR: Root-owned files detected!"
    echo "All files should be owned by the user running Docker."
    exit 1
fi

# Run quick validation (if available)
if [ -x "./scripts/validate-stack.sh" ]; then
    echo "Running stack validation..."
    if ! ./scripts/validate-stack.sh; then
        echo "ERROR: Stack validation failed!"
        echo "Fix issues before committing, or use 'git commit --no-verify' to skip."
        exit 1
    fi
fi

echo "Pre-commit validation passed!"
exit 0
```

Make executable: `chmod +x scripts/pre-commit`

#### 4.3 Create scripts/setup-hooks.sh

Generate hook installation script:
```bash
#!/usr/bin/env bash
set -euo pipefail

echo "Installing git hooks..."

# Check if .git directory exists
if [ ! -d ".git" ]; then
    echo "ERROR: Not a git repository!"
    exit 1
fi

# Install pre-commit hook
if [ -f "scripts/pre-commit" ]; then
    cp scripts/pre-commit .git/hooks/pre-commit
    chmod +x .git/hooks/pre-commit
    echo "✓ Installed pre-commit hook"
else
    echo "✗ scripts/pre-commit not found"
    exit 1
fi

echo "Git hooks installed successfully!"
```

Make executable: `chmod +x scripts/setup-hooks.sh`

**After creating scripts**: Run `./scripts/setup-hooks.sh` to install hooks.

### Phase 5: Docker Configuration

#### 5.1 Create docker-compose.yml Template

Generate initial docker-compose.yml based on user requirements:

**IMPORTANT**: Use the **docker-validation** skill to ensure the docker-compose.yml follows all best practices.

Basic template structure:
```yaml
services:
  # Services will be added based on requirements

  # Example: nginx service
  nginx:
    image: nginx:alpine
    container_name: ${PROJECT_NAME:-project}_nginx
    restart: unless-stopped
    ports:
      - "${NGINX_PORT:-80}:80"
    volumes:
      - ./config/nginx:/etc/nginx/conf.d:ro
    networks:
      - app-network

networks:
  app-network:
    driver: bridge

# Secrets will be defined here when needed
secrets: {}

# Volumes for persistent data
volumes: {}
```

**Actions**:
1. Generate docker-compose.yml with selected services
2. Use **docker-validation** skill to validate
3. Fix any issues reported
4. NEVER proceed if validation fails - ask user for guidance

#### 5.2 Create .env.example

Generate environment template:
```bash
# Project Configuration
PROJECT_NAME=myproject

# Service Ports
NGINX_PORT=80
# Add more as needed

# Database Configuration (if applicable)
# POSTGRES_PORT=5432
# REDIS_PORT=6379

# IMPORTANT: Copy this file to .env and configure for your environment
# .env is gitignored and should contain actual secrets
```

### Phase 6: Configuration Files

Use the **config-generator** skill to create service-specific configuration files:

**Actions**:
1. For each service (nginx, PostgreSQL, Redis), invoke config-generator
2. Generate configs in ./config/<service-name>/
3. Validate generated configs with config-generator validation
4. NEVER proceed with invalid configs - ask user for guidance

**Example invocation**:
- "Generate nginx production configuration for this stack"
- "Create PostgreSQL development configuration"
- "Generate Redis configuration with persistence"

### Phase 7: Secrets Management

Use the **secrets-manager** skill to set up secrets:

**Actions**:
1. Identify services that require secrets
2. Use secrets-manager to create secure secrets
3. Configure docker-compose.yml with secret references
4. Generate docker-entrypoint.sh scripts ONLY if containers don't support native Docker secrets
5. Validate with secrets-manager that NO secrets are in .env or docker-compose.yml environment
6. NEVER proceed if secrets-manager reports issues - ask user for guidance

**Example invocation**:
- "Set up database password secret for PostgreSQL"
- "Create Redis password secret"
- "Generate secure random secret for JWT_SECRET"

### Phase 8: Documentation Generation

#### 8.1 Create docs/setup.md

Generate setup documentation:
```markdown
# Setup Instructions

## Prerequisites

- Docker Engine 20.10+
- Docker Compose V2
- Git

## Initial Setup

1. Clone the repository:
   ```bash
   git clone <repository-url>
   cd <project-name>
   ```

2. Copy environment template:
   ```bash
   cp .env.example .env
   ```

3. Configure environment:
   - Edit .env with your settings
   - NEVER commit .env to git

4. Set up secrets:
   - Follow secrets/README.md for secret creation
   - Use `docker secret create` for each secret

5. Install git hooks:
   ```bash
   ./scripts/setup-hooks.sh
   ```

6. Validate stack:
   ```bash
   ./scripts/validate-stack.sh
   ```

7. Start services:
   ```bash
   docker compose up -d
   ```

## Validation

Always run validation before deploying:
```bash
./scripts/validate-stack.sh
```

This checks:
- Stack structure
- Secrets configuration
- Docker configuration
- File ownership
- Git exclusions
```

#### 8.2 Create docs/services.md

Document all services:
```markdown
# Services Documentation

## Service Overview

| Service | Port | Purpose | Configuration |
|---------|------|---------|---------------|
| nginx | 80 | Web server | ./config/nginx |
| ... | ... | ... | ... |

## Service Details

### Nginx

**Image**: nginx:alpine
**Purpose**: Web server and reverse proxy
**Configuration**: ./config/nginx/
**Secrets**: None
**Volumes**:
- ./config/nginx:/etc/nginx/conf.d:ro

### [Other Services]

Document each service similarly...
```

#### 8.3 Create docs/decisions/0001-stack-architecture.md

Create architecture decision record:
```markdown
# 1. Stack Architecture

**Date**: YYYY-MM-DD

## Status

Accepted

## Context

This project uses GitLab Stack Management patterns to ensure:
- Consistent configuration management
- Secure secrets handling
- Proper file ownership
- Validated deployments

## Decision

We will follow these principles:
1. All configuration in docker-compose.yml and ./config
2. All secrets in ./secrets and Docker secrets
3. docker-entrypoint.sh only when necessary
4. No root-owned files
5. ./_temporary for transient files
6. Complete validation before deployment

## Consequences

**Positive**:
- Consistent structure across all stacks
- Automated validation prevents issues
- Secure by default
- Easy to maintain and update

**Negative**:
- Initial setup requires more steps
- Must follow strict patterns
- Validation gates can slow rapid iteration

## Compliance

- stack-validator: Ensures structure compliance
- secrets-manager: Ensures secure secrets
- docker-validation: Ensures Docker best practices
```

#### 8.4 Create CLAUDE.md

Generate Claude Code instructions for the project:
```markdown
# CLAUDE.md

This file provides guidance to Claude Code when working with this stack project.

## Project Structure

This is a GitLab Stack project following strict patterns:
- Configuration: docker-compose.yml and ./config/
- Secrets: ./secrets/ and Docker secrets
- Temporary: ./_temporary/ (gitignored)
- Documentation: ./docs/
- Validation: ./scripts/

## Required Skills

This project uses these Claude Code skills:
- **stack-validator**: Validate entire stack structure
- **secrets-manager**: Manage Docker secrets securely
- **docker-validation**: Validate Docker configurations
- **config-generator**: Generate service configs

## Git Configuration

**Branch**: main
**Merge Strategy**: ff-only (fast-forward only)
**Hooks**: Pre-commit validation enabled

## Validation Requirements

BEFORE any git commit, ALL these must pass:
1. stack-validator reports NO issues
2. secrets-manager is satisfied
3. docker-validation is satisfied
4. No root-owned files exist
5. No secrets in .env or docker-compose.yml environment

## Making Changes

### Adding a Service

1. Update docker-compose.yml
2. Use config-generator to create configs
3. Use secrets-manager to handle secrets
4. Run ./scripts/validate-stack.sh
5. Fix ALL issues before committing
6. NEVER commit if validation fails

### Modifying Configuration

1. Edit configuration files
2. Validate with docker-validation
3. Run ./scripts/validate-stack.sh
4. Fix ALL issues before committing

### Working with Secrets

1. Use secrets-manager for ALL secret operations
2. NEVER put secrets in .env or docker-compose.yml
3. Use Docker secrets or ./secrets/ directory
4. Validate with secrets-manager before committing

## Commit Standards

**Pre-commit Hook**: Automatically validates before commit
**Skip Hook**: `git commit --no-verify` (emergency only)
**Commit Messages**: Use conventional commits format

Example:
```
feat: add Redis service with persistence
fix: correct nginx proxy configuration
docs: update setup instructions
```

## Important Rules

1. NEVER commit secrets to git
2. NEVER create root-owned files
3. NEVER skip validation without asking
4. NEVER use workarounds - ask user instead
5. ALWAYS run validation before committing
6. ALWAYS document decisions in ./docs/decisions/
7. ALWAYS use ff-only merge strategy
8. ALWAYS use main as branch name

## Troubleshooting

### Validation Fails

1. Review validation output
2. Use respective skill to investigate (stack-validator, secrets-manager, docker-validation)
3. Fix reported issues
4. Re-run validation
5. Ask user if stuck - NEVER use workarounds

### Git Conflicts

1. NEVER use rebase without explicit permission
2. Use ff-only merges
3. Ask user how to resolve conflicts

### Permission Issues

1. Check file ownership: `find . -user root`
2. Fix with: `sudo chown -R $USER:$USER .`
3. Validate with stack-validator

## Resources

- Stack Validator Documentation
- Secrets Manager Documentation
- Docker Validation Documentation
- Config Generator Documentation
```

#### 8.5 Create README.md

Generate project README:
```markdown
# Project Name

Brief description of the project.

## Quick Start

```bash
# Clone repository
git clone <url>
cd <project-name>

# Copy environment template
cp .env.example .env

# Configure environment (edit .env)
nano .env

# Install git hooks
./scripts/setup-hooks.sh

# Validate stack
./scripts/validate-stack.sh

# Start services
docker compose up -d
```

## Documentation

- [Setup Instructions](./docs/setup.md)
- [Services Documentation](./docs/services.md)
- [Architecture Decisions](./docs/decisions/)

## Project Structure

```
.
├── config/          # Service configurations
├── secrets/         # Docker secrets (gitignored except .gitkeep)
├── _temporary/      # Temporary files (gitignored)
├── scripts/         # Validation and utility scripts
├── docs/            # Project documentation
└── docker-compose.yml
```

## Validation

This project uses automated validation:

```bash
# Full validation
./scripts/validate-stack.sh

# Pre-commit validation (automatic)
git commit -m "message"
```

Validation checks:
- ✓ Stack structure (stack-validator)
- ✓ Secrets configuration (secrets-manager)
- ✓ Docker configuration (docker-validation)
- ✓ File ownership (no root files)
- ✓ Git exclusions (.gitignore)

## Git Workflow

**Branch**: main
**Merge Strategy**: ff-only (fast-forward only)
**Pre-commit Hook**: Enabled (validates before commit)

## Services

| Service | Port | Purpose |
|---------|------|---------|
| ... | ... | ... |

See [Services Documentation](./docs/services.md) for details.

## Development

### Prerequisites

- Docker Engine 20.10+
- Docker Compose V2
- Git
- Bash (for scripts)

### Adding a Service

1. Update docker-compose.yml
2. Generate configs: Use config-generator skill
3. Set up secrets: Use secrets-manager skill
4. Validate: `./scripts/validate-stack.sh`
5. Commit changes

### Making Changes

1. Make your changes
2. Run validation: `./scripts/validate-stack.sh`
3. Fix any issues reported
4. Commit (pre-commit hook will validate again)

## Troubleshooting

### Validation Fails

Run individual validators:
- `claude-code-cli run stack-validator`
- `claude-code-cli run secrets-manager --validate`
- `claude-code-cli run docker-validation`

### Permission Issues

Check for root-owned files:
```bash
find . -user root -not -path "./.git/*"
```

Fix ownership:
```bash
sudo chown -R $USER:$USER .
```

## License

[Your License]

## Contact

[Your Contact Info]
```

### Phase 9: Final Validation

#### 9.1 Run Complete Validation

Execute full validation workflow:

```bash
# Run validation script
./scripts/validate-stack.sh
```

This must check:
1. **stack-validator**: Full structure validation
2. **secrets-manager**: All secrets properly configured
3. **docker-validation**: Docker configs valid
4. **File ownership**: No root-owned files
5. **Git setup**: Proper .gitignore, hooks installed

#### 9.2 Verification Checklist

Manually verify:
- [ ] All directories created
- [ ] Git repository initialized
- [ ] Git configured (main branch, ff-only)
- [ ] .gitignore and .dockerignore created
- [ ] Validation scripts created and executable
- [ ] Git hooks installed
- [ ] docker-compose.yml created and valid
- [ ] .env.example created
- [ ] Service configs generated (if applicable)
- [ ] Secrets properly configured (if applicable)
- [ ] All documentation created
- [ ] CLAUDE.md created
- [ ] README.md created
- [ ] stack-validator: ✓ NO issues
- [ ] secrets-manager: ✓ NO issues
- [ ] docker-validation: ✓ NO issues

#### 9.3 Complete Stack Creation

**ONLY** mark stack creation as complete when:
- All validation passes with NO errors
- All documentation is complete
- Git repository is properly configured
- User confirms everything is correct

If ANY validation fails:
1. Report the issue clearly
2. Ask user how to proceed
3. NEVER use workarounds
4. Fix the issue properly
5. Re-validate

### Phase 10: Initial Commit

Once all validation passes:

```bash
# Stage all files
git add .

# Commit (pre-commit hook will validate)
git commit -m "feat: initial stack setup

- Created directory structure
- Configured git with main branch and ff-only
- Set up validation scripts and hooks
- Generated docker-compose.yml
- Created service configurations
- Set up secrets management
- Added comprehensive documentation"

# Push to remote (if configured)
git push -u origin main
```

If commit fails due to pre-commit hook:
1. Review validation output
2. Fix issues
3. Re-run validation
4. Try commit again
5. Ask user if issues persist

## Communication Style

When creating a stack:
- Ask clarifying questions for missing information
- Explain each phase before executing
- Report validation results clearly
- NEVER proceed if validation fails
- Ask user for guidance when stuck
- Confirm important decisions before executing
- Document all decisions in ./docs/decisions/
- Provide clear next steps

## Integration with Other Skills

### stack-validator

- Call at the end of Phase 9 (Final Validation)
- Call in validation scripts
- Call in pre-commit hooks
- NEVER proceed if validation fails

### secrets-manager

- Call during Phase 7 (Secrets Management)
- Use for ALL secret operations
- Validate secrets configuration
- NEVER proceed if secrets-manager reports issues

### docker-validation

- Call during Phase 5 (Docker Configuration)
- Validate docker-compose.yml
- Validate Dockerfiles (if any)
- Call in validation scripts
- NEVER proceed if docker-validation fails

### config-generator

- Call during Phase 6 (Configuration Files)
- Generate service-specific configs
- Use for nginx, PostgreSQL, Redis, etc.
- Validate generated configs

## Error Handling

### Validation Failure

**DO NOT**:
- Try workarounds
- Skip validation
- Proceed with errors
- Assume user wants to continue

**DO**:
- Stop immediately
- Report the issue clearly
- Show validation output
- Ask user how to proceed
- Document the decision

### Git Issues

**DO NOT**:
- Force push
- Skip hooks without permission
- Use rebase without permission
- Modify git history without permission

**DO**:
- Ask user for guidance
- Explain the issue
- Provide options
- Document the solution

### Permission Issues

**DO NOT**:
- Create root-owned files
- Use sudo without permission
- Skip permission checks

**DO**:
- Report permission issues
- Ask user to fix ownership
- Validate after fixing
- Document proper ownership

## Example Workflows

### Workflow 1: Creating a Simple Web Stack

```
User: "Create a new stack for a web application with nginx"
Assistant: "I'll create a new web stack with nginx. Let me gather some information first."

**Questions**:
1. "What's the project name?"
   User: "mywebapp"

2. "Do you have a remote git repository URL?"
   User: "https://gitlab.com/user/mywebapp.git"

3. "What environment is this for?"
   User: "production"

**Actions**:
1. Create directory structure
2. Initialize git and connect to remote
3. Configure git (main branch, ff-only)
4. Create docker-compose.yml with nginx
5. Use docker-validation to validate
6. Use config-generator to create nginx config
7. Create validation scripts
8. Install git hooks
9. Generate all documentation
10. Run full validation
11. Report status

**Completion**:
"Stack creation complete! All validations passed:
✓ stack-validator: NO issues
✓ secrets-manager: NO issues (no secrets needed)
✓ docker-validation: NO issues

Next steps:
1. Review generated files
2. Customize .env from .env.example
3. Run: docker compose up -d"
```

### Workflow 2: Creating a Full Application Stack

```
User: "Create a stack with nginx, PostgreSQL, and Redis"
Assistant: "I'll create a full application stack. Let me gather information."

**Questions**:
1. "Project name?" → "myapp"
2. "Remote repository?" → "Yes, https://gitlab.com/user/myapp.git"
3. "Environment?" → "production"

**Actions**:
1. Create directory structure with config/{nginx,postgres,redis}
2. Initialize git, connect remote, configure (main, ff-only)
3. Create docker-compose.yml with all three services
4. Validate with docker-validation
5. Use config-generator for each service:
   - nginx production config
   - PostgreSQL production config
   - Redis production config
6. Use secrets-manager for:
   - PostgreSQL password
   - Redis password
   - Create Docker secrets
7. Create validation scripts and install hooks
8. Generate all documentation
9. Run full validation (stack-validator, secrets-manager, docker-validation)
10. Initial commit

**Completion**:
"Stack creation complete! All validations passed:
✓ stack-validator: NO issues
✓ secrets-manager: All secrets properly configured
✓ docker-validation: NO issues

Services configured:
- nginx (port 80)
- PostgreSQL (port 5432, password in Docker secret)
- Redis (port 6379, password in Docker secret)

Next steps:
1. Review secrets in ./secrets/
2. Customize .env from .env.example
3. Run: docker compose up -d"
```

## Summary

The stack-creator skill:

1. **Integrates** with stack-validator, secrets-manager, docker-validation, config-generator
2. **Enforces** complete validation before considering creation complete
3. **NEVER** uses workarounds - always asks user for guidance
4. **Configures** git properly (main branch, ff-only merges)
5. **Installs** validation hooks and scripts
6. **Documents** everything in ./docs/
7. **Creates** production-ready stacks following GitLab Stack Management patterns

Stack creation is complete ONLY when all three validators pass with NO issues and all documentation is in place.
