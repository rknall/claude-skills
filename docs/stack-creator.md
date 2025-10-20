# GitLab Stack Creator Skill

## Overview

The stack-creator skill helps create new GitLab stack projects from scratch or templates, ensuring they follow all architectural patterns and best practices from the start.

## Core Creation Principles

Creates stacks that follow:
- All configuration through docker-compose.yml and ./config directory
- Secrets in ./secrets and docker secrets
- docker-entrypoint.sh only when containers don't support native secrets
- No root-owned files (all files owned by current user)
- _temporary directory for transient files
- Proper .gitignore for secrets and temporary files

## Stack Structure Created

```
project-name/
├── docker-compose.yml
├── .gitignore
├── .env.example
├── README.md
├── setup.sh (optional)
├── config/
│   └── .gitkeep
├── secrets/
│   └── .gitkeep
└── _temporary/
    └── .gitkeep
```

## Features

### 1. Interactive Stack Creation
Prompts for:
- Stack name
- Services to include (from templates)
- Network configuration
- Volume requirements
- Secret requirements

### 2. Template-Based Creation
Pre-built templates for common stacks:
- **web-stack**: nginx + app + postgres
- **monitoring-stack**: prometheus + grafana
- **database-stack**: postgres/mysql + redis
- **gitlab-stack**: gitlab + runner + registry
- **media-stack**: jellyfin/plex + *arr stack
- **custom**: build from scratch

### 3. Automatic Best Practices
- Generates docker-compose.yml with proper secret references
- Creates .gitignore excluding secrets and _temporary
- Sets up proper directory permissions
- Initializes git repository (optional)
- Creates template README with documentation
- Generates .env.example for non-secret configuration

### 4. docker-entrypoint.sh Generation
When services don't support docker secrets natively:
- Creates docker-entrypoint.sh script
- Includes secret loading logic
- Sets proper permissions (755)
- Documents when to use vs not use

## Usage Examples

### Interactive Mode
```bash
# Create new stack interactively
claude create-stack

# Create in specific directory
claude create-stack /path/to/new-stack

# Create with specific template
claude create-stack --template web-stack

# Create with name directly
claude create-stack my-awesome-stack
```

### Template Mode
```bash
# Create from specific template
claude create-stack --template gitlab-stack

# List available templates
claude create-stack --list-templates

# Create with services
claude create-stack --services nginx,postgres,redis

# Create minimal stack (no templates)
claude create-stack --minimal
```

### Advanced Options
```bash
# Create without git initialization
claude create-stack --no-git

# Create with custom user/group
claude create-stack --user 1001:1001

# Create with validation
claude create-stack --validate

# Dry run (show what would be created)
claude create-stack --dry-run
```

## docker-compose.yml Template

Generated compose file follows best practices:

```yaml
version: '3.8'

services:
  app:
    image: nginx:alpine
    container_name: ${STACK_NAME}_app
    restart: unless-stopped
    user: "${UID}:${GID}"
    ports:
      - "${APP_PORT:-8080}:80"
    volumes:
      - ./config/nginx.conf:/etc/nginx/nginx.conf:ro
      - app_data:/var/www/html
    secrets:
      - app_secret
    networks:
      - stack_network

secrets:
  app_secret:
    file: ./secrets/app_secret.txt

volumes:
  app_data:
    driver: local

networks:
  stack_network:
    driver: bridge
```

## .gitignore Template

```gitignore
# Secrets - NEVER commit these
/secrets/*
!secrets/.gitkeep

# Temporary files
/_temporary/*
!_temporary/.gitkeep

# Environment files with secrets
.env

# Docker volumes (if using local bind mounts)
/volumes/

# OS files
.DS_Store
Thumbs.db

# IDE
.idea/
.vscode/
*.swp
*.swo

# Logs
*.log
```

## README Template

Generated README includes:

```markdown
# [Stack Name]

## Overview
[Description of what this stack does]

## Services
- **service1**: Description and port
- **service2**: Description and port

## Prerequisites
- Docker and Docker Compose
- User must have permission to run Docker

## Setup

1. Copy environment template:
   \`\`\`bash
   cp .env.example .env
   \`\`\`

2. Configure secrets in ./secrets/:
   - Create `./secrets/secret_name.txt` for each required secret

3. Review and customize ./config/ files

4. Start the stack:
   \`\`\`bash
   docker-compose up -d
   \`\`\`

## Configuration

Configuration files are in `./config/`:
- List of config files and their purpose

## Secrets

This stack requires the following secrets in `./secrets/`:
- `secret_name.txt`: Description

**IMPORTANT**: Never commit files in ./secrets/ directory!

## Maintenance

### Logs
\`\`\`bash
docker-compose logs -f [service]
\`\`\`

### Restart
\`\`\`bash
docker-compose restart [service]
\`\`\`

### Update
\`\`\`bash
docker-compose pull
docker-compose up -d
\`\`\`

## Troubleshooting

Common issues and solutions...

## Architecture

[Diagram or description of how services connect]
```

## Service Templates

### nginx Service
```yaml
nginx:
  image: nginx:alpine
  user: "${UID}:${GID}"
  volumes:
    - ./config/nginx.conf:/etc/nginx/nginx.conf:ro
  ports:
    - "${NGINX_PORT:-80}:80"
```

### PostgreSQL Service
```yaml
postgres:
  image: postgres:15-alpine
  user: "${UID}:${GID}"
  environment:
    POSTGRES_DB: ${POSTGRES_DB}
    POSTGRES_USER: ${POSTGRES_USER}
  secrets:
    - db_password
  volumes:
    - postgres_data:/var/lib/postgresql/data
```

### Application with docker-entrypoint.sh
```yaml
app:
  image: myapp:latest
  user: "${UID}:${GID}"
  entrypoint: ["/docker-entrypoint.sh"]
  secrets:
    - api_key
  volumes:
    - ./docker-entrypoint.sh:/docker-entrypoint.sh:ro
```

## docker-entrypoint.sh Template

For services that need it:

```bash
#!/bin/bash
set -e

# Load secrets from docker secrets
if [ -f /run/secrets/api_key ]; then
  export API_KEY=$(cat /run/secrets/api_key)
fi

if [ -f /run/secrets/db_password ]; then
  export DB_PASSWORD=$(cat /run/secrets/db_password)
fi

# Execute the main command
exec "$@"
```

## Post-Creation Actions

After creating stack, Claude should:

1. **Set Permissions**: Ensure all files owned by current user
2. **Validate**: Run stack-validator on created stack
3. **Initialize Git** (if requested): 
   - `git init`
   - Add initial commit
   - Create .git/hooks for secret protection
4. **Show Next Steps**: Display what user needs to do next
5. **Clean _temporary**: Remove any temp files used during creation

## Next Steps Output

```
✅ Stack created successfully: my-stack

📋 Next Steps:
1. Review docker-compose.yml and customize as needed
2. Create required secrets in ./secrets/:
   - ./secrets/db_password.txt
   - ./secrets/api_key.txt
3. Copy and configure .env file:
   cp .env.example .env
4. Review configuration files in ./config/
5. Start your stack:
   docker-compose up -d

📚 Documentation: ./README.md
🔍 Validate: claude validate-stack ./my-stack
```

## Integration with Other Skills

- **stack-validator**: Automatically validate after creation
- **secrets-manager**: Help generate initial secrets
- **documentation-generator**: Generate extended docs
- **setup script**: Include reference to setup.sh if present

## Best Practices Enforced

1. **No hardcoded secrets**: All secrets use docker secrets
2. **User ownership**: All files owned by current user (UID:GID)
3. **Proper .gitignore**: Secrets and temp files excluded
4. **Documentation**: README with clear setup steps
5. **Example configs**: .env.example for reference
6. **Minimal docker-entrypoint.sh**: Only when necessary
7. **Standard structure**: Consistent across all stacks

## Error Handling

- Check if directory already exists
- Validate template selection
- Ensure Docker is installed
- Verify user has necessary permissions
- Validate generated files before finalizing
- Rollback on failure

## Configuration

Allow .stack-creator.yml for defaults:
```yaml
# Default settings
default_template: web-stack
default_user: "${UID}:${GID}"
auto_validate: true
init_git: true
templates_path: ~/.stack-templates/
```

## Custom Templates

Users can add custom templates to `~/.stack-templates/`:
```
~/.stack-templates/
├── my-template/
│   ├── template.yml (describes template)
│   ├── docker-compose.yml
│   ├── config/
│   └── README.md
```

## Validation on Creation

After creation, automatically:
- Check directory structure
- Validate docker-compose.yml syntax
- Verify .gitignore coverage
- Confirm file ownership
- Test stack can be parsed

---

*This skill ensures all new stacks start with best practices built-in.*

## Version 1.0.0 Implementation Details

**Created:** 2025-10-20
**Phase:** 1 (Complete)
**Status:** Production Ready

### Implementation Summary

This version implements the complete stack-creator skill as specified in ideas.md Phase 1, with full integration of all Phase 1 skills.

### Complete Feature Set

1. **10-Phase Creation Workflow**
   - Phase 1: Gather Requirements
   - Phase 2: Directory Structure Creation
   - Phase 3: Git Repository Setup
   - Phase 4: Validation Scripts Setup
   - Phase 5: Docker Configuration
   - Phase 6: Configuration Files
   - Phase 7: Secrets Management
   - Phase 8: Documentation Generation
   - Phase 9: Final Validation
   - Phase 10: Initial Commit

2. **Git Configuration**
   - main as default branch
   - ff-only merge strategy
   - Pre-commit validation hooks
   - Comprehensive .gitignore

3. **Validation Integration**
   - stack-validator integration
   - secrets-manager integration
   - docker-validation integration (via config-generator)
   - config-generator integration

4. **Documentation Generation**
   - README.md
   - CLAUDE.md
   - docs/setup.md
   - docs/services.md
   - docs/decisions/0001-stack-architecture.md

5. **Validation Scripts**
   - scripts/validate-stack.sh (comprehensive)
   - scripts/pre-commit (git hook)
   - scripts/setup-hooks.sh (installer)
   - scripts/pre-push (optional)

### Completion Criteria

Stack creation is complete ONLY when:
- ✅ stack-validator: NO issues
- ✅ secrets-manager: Satisfied, NO open issues
- ✅ docker-validation: NO issues
- ✅ All validation scripts execute successfully
- ✅ Git repository properly initialized and configured
- ✅ Documentation complete in ./docs

### Error Handling Policy

**NO WORKAROUNDS EVER**
- If something fails, STOP and ask user
- Never assume or guess
- Never skip validation
- Never force operations
- Always provide clear error messages
- Always offer options

### Phase 1 Completion

With this implementation, Phase 1 is **COMPLETE**:
1. ✅ stack-validator (v1.0.0)
2. ✅ stack-creator (v1.0.0) ← THIS SKILL
3. ✅ secrets-manager (v1.0.0)
4. ✅ config-generator (v1.0.0)

All four priority skills are implemented and integrated.

### Files Delivered

- SKILL.md (1,088 lines)
- README.md (421 lines)
- git-hooks-guide.md (736 lines)
- templates-reference.md (1,015 lines)
- workflow-examples.md

Total: 3,260+ lines of comprehensive documentation and implementation guidance.

### Integration Success

The skill successfully integrates all Phase 1 skills:
- Calls stack-validator for structure validation
- Calls secrets-manager for secret operations
- Calls docker-validation (via config-generator) for Docker configs
- Calls config-generator for service configs

### Next Phases

Phase 2 skills (future):
- stack-debugger
- cleanup-manager

Phase 3 skills (future):
- migration-helper

Phase 4 skills (future):
- documentation-generator

---

**Phase 1 Status: COMPLETE ✅**
**Production Ready: YES ✅**
**All Validators Integration: YES ✅**
**No-Workaround Policy: ENFORCED ✅**

