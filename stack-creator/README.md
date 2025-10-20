# GitLab Stack Creator

Create new GitLab stack projects with proper directory structure, git configuration, validation hooks, and comprehensive documentation.

## Overview

The Stack Creator skill automates the creation of new GitLab stack projects following strict best practices. It integrates with stack-validator, secrets-manager, and docker-validation skills to ensure complete, validated stack setups.

## When to Use

Use this skill when you need to:
- Create a new GitLab stack project from scratch
- Initialize a Docker stack with proper directory structure
- Set up git repository with validation hooks
- Bootstrap a project with automated validation
- Create a stack following GitLab Stack Management patterns

## Installation

```bash
/plugin install stack-creator
```

## Core Principles

Stack Creator follows GitLab Stack Management patterns:

1. **Everything in docker-compose.yml and ./config**
2. **Secrets in ./secrets and Docker secrets**
3. **docker-entrypoint.sh only when necessary**
4. **No root-owned files**
5. **./_temporary for transient files**
6. **Complete validation before completion**

## Stack Creation is Complete When

A stack is considered complete ONLY when:

- ✅ **stack-validator** reports NO issues
- ✅ **secrets-manager** is satisfied (NO open issues)
- ✅ **docker-validation** is satisfied (NO issues)
- ✅ All validation scripts execute successfully
- ✅ Git repository properly initialized and configured
- ✅ Documentation complete in ./docs

**IMPORTANT**: This skill NEVER uses workarounds. If something fails, it stops and asks for user guidance.

## Usage

### Triggering the Skill

The skill activates when you mention:
- "Create a new stack"
- "Initialize a GitLab stack"
- "Set up a Docker stack project"
- "Create stack with [services]"

### Example Prompts

1. **Simple Web Stack**:
   ```
   Create a new stack for a web application with nginx
   ```

2. **Full Application Stack**:
   ```
   Create a stack with nginx, PostgreSQL, and Redis for production
   ```

3. **Development Stack**:
   ```
   Initialize a development stack with PostgreSQL and Redis
   ```

## Generated Structure

```
project-name/
├── .git/                       # Git repository
│   ├── hooks/                  # Git hooks (validation)
│   └── config                  # Git configuration
├── config/                     # Service configurations
│   ├── nginx/                  # Nginx configs
│   ├── postgres/               # PostgreSQL configs
│   └── redis/                  # Redis configs
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

## Features

### Git Repository Setup

- Initializes or connects to remote repository
- Configures main branch as default
- Sets ff-only merge strategy
- Creates comprehensive .gitignore
- Installs pre-commit validation hooks

### Docker Configuration

- Generates docker-compose.yml with best practices
- Validates with docker-validation skill
- Creates .env.example template
- Ensures proper service configuration

### Secrets Management

- Integrates with secrets-manager skill
- Sets up ./secrets directory properly
- Ensures NO secrets in .env or docker-compose.yml
- Generates docker-entrypoint.sh only when needed

### Service Configuration

- Uses config-generator for service configs
- Creates configs in ./config/<service-name>/
- Validates all generated configurations
- Supports nginx, PostgreSQL, Redis, and more

### Validation Scripts

- **validate-stack.sh**: Complete stack validation
- **pre-commit**: Pre-commit validation hook
- **setup-hooks.sh**: Git hooks installation

All scripts executable and ready to use.

### Documentation

- **docs/setup.md**: Setup instructions
- **docs/services.md**: Service documentation
- **docs/decisions/**: Architecture decision records
- **CLAUDE.md**: Claude Code project instructions
- **README.md**: Project overview

## Workflow

### Phase 1: Gather Requirements

Asks for:
- Project name
- Services needed
- Remote git repository URL (optional)
- Environment type
- Special requirements

### Phase 2: Create Structure

Creates:
- Directory structure
- Git repository
- Git configuration (main, ff-only)
- .gitignore and .dockerignore

### Phase 3: Docker Setup

Generates:
- docker-compose.yml
- .env.example
- Validates with docker-validation

### Phase 4: Service Configuration

Uses config-generator to create:
- nginx configuration
- PostgreSQL configuration
- Redis configuration
- Other service configs

### Phase 5: Secrets Setup

Uses secrets-manager to:
- Create secure secrets
- Configure Docker secrets
- Validate NO secrets in wrong places
- Generate entrypoint scripts if needed

### Phase 6: Validation Scripts

Creates:
- scripts/validate-stack.sh
- scripts/pre-commit
- scripts/setup-hooks.sh
- Installs hooks

### Phase 7: Documentation

Generates:
- docs/setup.md
- docs/services.md
- docs/decisions/0001-stack-architecture.md
- CLAUDE.md
- README.md

### Phase 8: Final Validation

Runs complete validation:
- stack-validator
- secrets-manager
- docker-validation
- File ownership check
- Git setup verification

### Phase 9: Initial Commit

Creates initial commit when validation passes.

## Validation

### Pre-Commit Hook

Automatically runs before each commit:
- Checks for secrets in staged files
- Checks for root-owned files
- Runs full stack validation
- Blocks commit if issues found

Can skip with: `git commit --no-verify` (emergency only)

### Manual Validation

Run anytime:
```bash
./scripts/validate-stack.sh
```

Validates:
- Stack structure
- Secrets configuration
- Docker configuration
- File ownership
- Git exclusions

## Git Configuration

### Branch Strategy

- **Default Branch**: main
- **Merge Strategy**: ff-only (fast-forward only)
- **No Rebase**: Unless explicitly requested

### Commit Standards

Example commit messages:
```
feat: add Redis service with persistence
fix: correct nginx proxy configuration
docs: update setup instructions
chore: update validation scripts
```

## Integration with Other Skills

### stack-validator

- Called during final validation
- Called in validation scripts
- Called in pre-commit hooks
- Must pass with NO issues

### secrets-manager

- Called during secrets setup
- Validates secret configuration
- Ensures NO secrets in .env
- Must be satisfied before completion

### docker-validation

- Called during Docker setup
- Validates docker-compose.yml
- Validates Dockerfiles
- Must pass with NO issues

### config-generator

- Called for each service
- Generates service configs
- Validates generated configs
- Creates configs in ./config/

## Error Handling

### Validation Failures

When validation fails:
1. Stops immediately
2. Reports issue clearly
3. Shows validation output
4. Asks user how to proceed
5. NEVER uses workarounds

### Git Issues

When git operations fail:
1. Reports the error
2. Explains the issue
3. Asks for user guidance
4. Documents the solution

### Permission Issues

When permission problems occur:
1. Reports root-owned files
2. Asks user to fix ownership
3. Validates after fixing
4. Documents proper ownership

## Best Practices

1. **Always validate** before committing
2. **Never skip hooks** without good reason
3. **Document decisions** in ./docs/decisions/
4. **Use ff-only merges** for clean history
5. **Keep secrets secure** - never commit
6. **Fix ownership** - no root files
7. **Ask when stuck** - no workarounds
8. **Follow validation** - fix all issues

## Example: Creating a Web Stack

```
User: "Create a new stack for mywebapp with nginx"
Stack Creator Response:
"I'll create mywebapp stack with nginx. Let me ask a few questions..."

Result:
✓ Directory structure created
✓ Git repository initialized (main, ff-only)
✓ docker-compose.yml created and validated
✓ nginx configuration generated
✓ Validation scripts installed
✓ Git hooks configured
✓ Documentation complete

All validations passed!
```

## Troubleshooting

### Stack Creation Fails

If stack creation stops:
1. Review the error message
2. Fix the reported issue
3. Ask the skill to continue
4. NEVER skip validation

### Validation Won't Pass

If validation keeps failing:
1. Run validators individually:
   - `claude-code run stack-validator`
   - `claude-code run secrets-manager --validate`
   - `claude-code run docker-validation`
2. Fix reported issues one by one
3. Re-run validation
4. Ask for help if stuck

### Git Configuration Issues

If git setup fails:
1. Check git is installed: `git --version`
2. Check git config: `git config --list`
3. Set manually if needed:
   ```bash
   git config init.defaultBranch main
   git config merge.ff only
   git config pull.ff only
   ```

### Permission Problems

If root-owned files appear:
```bash
# Find them
find . -user root -not -path "./.git/*"

# Fix ownership
sudo chown -R $USER:$USER .

# Re-validate
./scripts/validate-stack.sh
```

## Version History

- **v0.1.0** (2025-10-20): Initial release
  - Complete Phase 1 stack creation
  - Integration with stack-validator, secrets-manager, docker-validation, config-generator
  - Git repository setup with validation hooks
  - Comprehensive documentation generation
  - No-workaround policy

## Resources

- [stack-validator Documentation](../stack-validator/README.md)
- [secrets-manager Documentation](../secrets-manager/README.md)
- [docker-validation Documentation](../docker-validation/README.md)
- [config-generator Documentation](../config-generator/README.md)
- [Git Hooks Guide](./git-hooks-guide.md)

## License

This skill follows the same license as the Skills marketplace repository.
