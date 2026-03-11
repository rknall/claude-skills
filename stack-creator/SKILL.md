---
name: "stack-creator"
description: "Create new GitLab stack projects from templates with proper directory structure, git configuration, validation hooks, and documentation. Use when initializing new Docker stack projects that follow GitLab stack patterns."
---

# GitLab Stack Creator

Expert assistance for creating new GitLab stack projects with proper structure, git configuration, validation scripts, and comprehensive documentation.

## When to Use This Skill

- Creating a new GitLab stack project
- Initializing a Docker stack with proper structure
- Setting up a project with ./config, ./secrets, ./_temporary directories
- Configuring git repository for stack projects
- Setting up validation hooks and scripts

## Core Principles

1. **Everything configured through docker-compose.yml and ./config**
2. **Secrets stored in ./secrets and Docker secrets**
3. **docker-entrypoint.sh scripts only when containers don't support native secrets**
4. **All container files owned by the user running docker (no root-owned files)**
5. **./_temporary directory for transient setup files (cleaned up after use)**
6. **Complete validation before considering stack creation complete**

## Validation Gate

Every phase that produces artifacts must pass validation before proceeding. A stack is complete ONLY when all gates pass:

| Gate | Tool | Criteria |
|------|------|----------|
| Structure | stack-validator | NO issues reported |
| Secrets | secrets-manager | NO open issues |
| Docker | docker-validation | NO issues reported |
| Scripts | `./scripts/validate-stack.sh` | Exit code 0 |
| Git | Manual check | Repository initialized, hooks installed |
| Docs | Manual check | ./docs complete |

**If ANY gate fails**: STOP, report the issue, and ask the user how to proceed. NEVER use workarounds.

## Stack Creation Workflow

### Phase 1: Project Initialization

#### 1.1 Gather Requirements

Ask the user for:
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
├── .git/hooks/              # Git hooks (validation scripts)
├── config/                  # Service configurations
├── secrets/                 # Docker secrets files
├── _temporary/              # Temporary files (gitignored)
├── scripts/                 # Validation and utility scripts
├── docs/decisions/          # Architecture decision records
├── docker-compose.yml       # Main compose file
├── .env.example             # Environment template
├── .gitignore               # Git exclusions
├── .dockerignore            # Docker exclusions
├── CLAUDE.md                # Claude Code instructions
└── README.md                # Project overview
```

**Actions**:
1. Create directories: `mkdir -p config secrets _temporary scripts docs/decisions`
2. Create placeholder files: `touch secrets/.gitkeep`
3. Set proper permissions: `chmod 700 secrets`

### Phase 3: Git Repository Setup

#### 3.1 Initialize or Connect Repository

- **No repo exists**: `git init`, set `init.defaultBranch main`, optionally add remote
- **Remote URL provided**: Clone or `git remote add origin <url>`, verify with `git remote -v`
- **If setup fails**: STOP and ask user. NEVER attempt workarounds.

#### 3.2 Configure Git

```bash
git config init.defaultBranch main
git config pull.ff only
git config merge.ff only
git config pull.rebase false
```

Set `user.name` and `user.email` if not set globally.

#### 3.3 Create .gitignore and .dockerignore

See [templates-reference.md](./templates-reference.md) for standard `.gitignore` and `.dockerignore` templates. Key rules:
- `secrets/*` (except `.gitkeep`) NEVER committed
- `.env` and `_temporary/` always excluded
- Docker build context excludes `.git`, `docs/`, `secrets/`

### Phase 4: Validation Scripts and Git Hooks

Create the following scripts in `./scripts/` and install git hooks. See [git-hooks-guide.md](./git-hooks-guide.md) for full script templates.

| Script | Purpose |
|--------|---------|
| `validate-stack.sh` | Runs stack-validator, secrets-manager, and docker-validation; returns exit 0 only if all pass |
| `pre-commit` | Blocks commits containing secrets or root-owned files; runs validate-stack.sh |
| `setup-hooks.sh` | Copies pre-commit to `.git/hooks/` and sets executable |

**After creating scripts**: `chmod +x scripts/*.sh scripts/pre-commit && ./scripts/setup-hooks.sh`

### Phase 5: Docker Configuration

Generate `docker-compose.yml` based on user requirements. See [templates-reference.md](./templates-reference.md) for service templates.

**Actions**:
1. Generate docker-compose.yml with selected services
2. Validate with **docker-validation** skill
3. Fix any issues reported
4. Create `.env.example` with project-specific variables
5. Pass the [Validation Gate](#validation-gate) before proceeding

### Phase 6: Configuration Files

Use the **config-generator** skill to create service-specific configs in `./config/<service-name>/`. Validate generated configs before proceeding.

### Phase 7: Secrets Management

Use the **secrets-manager** skill:
1. Identify services that require secrets
2. Create secure secrets in `./secrets/`
3. Configure docker-compose.yml with secret references
4. Generate `docker-entrypoint.sh` ONLY if containers don't support native Docker secrets
5. Pass the [Validation Gate](#validation-gate) — no secrets in `.env` or `docker-compose.yml` environment

### Phase 8: Documentation Generation

Generate the following documentation. See [templates-reference.md](./templates-reference.md) for document templates.

| Document | Content |
|----------|---------|
| `docs/setup.md` | Prerequisites, initial setup steps, validation instructions |
| `docs/services.md` | Service overview table, per-service details (image, ports, configs, secrets, volumes) |
| `docs/decisions/0001-stack-architecture.md` | ADR documenting stack architecture choices |
| `CLAUDE.md` | Claude Code project instructions: structure, required skills, git config, validation, commit standards |
| `README.md` | Quick start, project structure, validation commands, service table |

### Phase 9: Final Validation

Run the complete [Validation Gate](#validation-gate):

```bash
./scripts/validate-stack.sh
```

Verify all checklist items:
- All directories and files created
- Git configured (main branch, ff-only, hooks installed)
- docker-compose.yml valid
- Secrets properly configured
- All documentation complete
- stack-validator, secrets-manager, docker-validation: ALL pass with NO issues

**ONLY** mark stack creation complete when all gates pass and user confirms.

### Phase 10: Initial Commit

Once all validation passes:

```bash
git add .
git commit -m "feat: initial stack setup

- Created directory structure
- Configured git with main branch and ff-only
- Set up validation scripts and hooks
- Generated docker-compose.yml
- Created service configurations
- Set up secrets management
- Added comprehensive documentation"

git push -u origin main
```

If commit fails due to pre-commit hook: review output, fix issues, re-validate, retry. Ask user if issues persist.

## Supporting Resources

| Resource | Description |
|----------|-------------|
| [templates-reference.md](./templates-reference.md) | Docker Compose, .env, .gitignore, .dockerignore, and documentation templates |
| [git-hooks-guide.md](./git-hooks-guide.md) | Validation scripts, pre-commit hooks, and hook installation |
| [workflow-examples.md](./workflow-examples.md) | Step-by-step example workflows for common stack types |
