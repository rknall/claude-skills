---
name: "docker-validation"
description: "Comprehensive Docker and Docker Compose validation following best practices and security standards. Use this skill when users ask to validate Dockerfiles, review Docker configurations, check Docker Compose files, verify multi-stage builds, audit Docker security, or ensure compliance with Docker best practices. Validates syntax, security, multi-stage builds, and modern Docker Compose requirements."
---

# Docker Configuration Validator

Validate Dockerfiles and Docker Compose files for best practices, security, and modern syntax compliance.

## When to Use This Skill

Activate when the user requests:
- Validate Dockerfiles or Docker Compose files
- Review Docker configurations for best practices
- Check for Docker security issues
- Verify multi-stage build implementation
- Audit Docker setup for production readiness
- Ensure modern Docker Compose syntax compliance
- Create validation scripts or CI/CD integration

## Core Workflow

### Phase 1: Initial Assessment

Determine the validation scope:

1. **What to validate** — Single/multiple Dockerfiles, Compose files, entire project directory
2. **Requirements** — Security compliance level, multi-stage build needs, CI/CD integration
3. **Available tools** — Check for Hadolint, DCLint, Docker CLI; install if needed (see [tool-installation.md](./tool-installation.md))

### Phase 2: Dockerfile Validation

#### Locate and lint Dockerfiles

```bash
# Find all Dockerfiles
find . -type f \( -name "Dockerfile*" ! -name "*.md" \)

# Hadolint validation (if available)
hadolint Dockerfile
hadolint --format json Dockerfile
hadolint --failure-threshold error Dockerfile
```

If Hadolint is unavailable, perform manual validation using the checklist below.

#### Manual Dockerfile checks

Validate against the full [validation-checklist.md](./validation-checklist.md). Key checks:

- **Base images**: No `:latest` tags, specific versions pinned, minimal base images (alpine/slim)
- **Multi-stage builds**: >= 2 FROM statements, named stages with `AS`, `COPY --from=` for artifacts
- **Security**: USER directive present (non-root), no secrets in Dockerfile, absolute WORKDIR paths
- **Layer optimization**: Combined RUN commands, package cache cleaned, .dockerignore exists
- **Best practices**: HEALTHCHECK defined, COPY over ADD, CMD/ENTRYPOINT in JSON notation, labels included

```bash
# Quick multi-stage build verification
FROM_COUNT=$(grep -c "^FROM " Dockerfile)
NAMED_STAGES=$(grep -c "^FROM .* AS " Dockerfile)
COPY_FROM=$(grep -c "COPY --from=" Dockerfile)
```

#### Classify findings by severity

| Severity | Examples |
|----------|---------|
| **CRITICAL** | Running as root, `:latest` tags, exposed secrets, invalid syntax |
| **HIGH** | No multi-stage build, no HEALTHCHECK, unpinned packages |
| **MEDIUM** | ADD instead of COPY, missing cache cleanup, missing labels |
| **LOW** | Could use slimmer base, could combine RUN commands |

### Phase 3: Docker Compose Validation

#### Locate and validate Compose files

```bash
find . -maxdepth 3 -type f \( -name "docker-compose*.yml" -o -name "docker-compose*.yaml" -o -name "compose*.yml" \)

# Check for obsolete version field (CRITICAL — removed in Compose v2.27.0+)
grep -q "^version:" docker-compose.yml && echo "ERROR: Remove obsolete version field"

# Built-in syntax validation
docker compose config --quiet
docker compose -f docker-compose.prod.yml config --quiet
```

#### DCLint validation (if available)

```bash
dclint docker-compose.yml
dclint --fix docker-compose.yml
```

#### Manual Compose checks

Validate against the Compose section of [validation-checklist.md](./validation-checklist.md). Key checks:

- **No `version:` field** — obsolete since Compose v2.27.0
- **No `:latest` tags** on images
- **Restart policies** defined on all services
- **Health checks** configured for critical services
- **Custom networks** defined with proper isolation
- **Named volumes** for persistence
- **No hardcoded secrets** — use `.env` files or Docker secrets
- **No `privileged: true`** unless absolutely necessary

### Phase 4: Multi-Stage Build Deep Dive

Verify multi-stage builds follow this pattern:

```dockerfile
# Build stage — full toolchain
FROM node:20-bullseye AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build && npm run test

# Production stage — minimal image
FROM node:20-alpine AS runtime
WORKDIR /app
COPY --from=builder /app/dist ./dist
USER node
CMD ["node", "dist/index.js"]
```

**Validate**: At least 2 named stages, `COPY --from=` transfers artifacts, final stage uses minimal base, build tools absent from final stage, final stage runs as non-root user.

### Phase 5: Security Audit

```bash
# Check for USER directive
USER_COUNT=$(grep -c "^USER " Dockerfile)
LAST_USER=$(grep "^USER " Dockerfile | tail -1 | awk '{print $2}')

if [ "$USER_COUNT" -eq 0 ]; then
    echo "CRITICAL: No USER specified (runs as root!)"
elif [ "$LAST_USER" == "root" ] || [ "$LAST_USER" == "0" ]; then
    echo "CRITICAL: Final USER is root"
fi
```

Additional security checks:
- Scan for passwords/API keys in ENV, COPY/ADD, and RUN commands
- Check base image age and support status — recommend Trivy or Snyk scanning
- Review EXPOSE directives for unnecessary port exposure
- Validate port mappings in Compose files

### Phase 6: Generate Validation Report

Structure the report with:

1. **Executive Summary** — File counts, issue counts by severity, overall PASS/WARNINGS/FAIL status
2. **Dockerfile Analysis** — Per-file results with line-specific issues and fixes
3. **Docker Compose Analysis** — Modern syntax compliance, per-service findings
4. **Recommendations** — Prioritized by Critical > High > Medium > Low with estimated effort

For each issue found, provide:
- Exact line number and current code
- Corrected code
- Impact explanation

## Deliverables

Provide based on user needs:

1. **Validation Report** — Executive summary, per-file findings, prioritized recommendations
2. **Fixed Configuration Files** — Corrected Dockerfiles, updated Compose files, `.hadolint.yaml`, `.dclintrc.json`
3. **Validation Script** (if requested) — Automated bash script with CI/CD exit codes
4. **CI/CD Integration** (if requested) — GitHub Actions or GitLab CI workflows

For tool installation instructions, see [tool-installation.md](./tool-installation.md).
For the complete validation checklist, see [validation-checklist.md](./validation-checklist.md).

## Key Rules Reference

| Area | Critical Rules |
|------|---------------|
| **Dockerfile** | No `:latest` (DL3006), non-root USER (DL3002), absolute WORKDIR (DL3000), pin packages (DL3008/DL3013), clean cache (DL3009) |
| **Compose** | No `version:` field, no `:latest` tags, restart policies, health checks, named volumes |
| **Multi-Stage** | Min 2 named stages, `COPY --from=` artifacts, minimal final base, no build tools in final |
| **Security** | Non-root USER, no secrets in images, minimal base images, least privilege |
