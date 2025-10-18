# Docker Validation Checklist

Complete checklist for validating Docker configurations. Use this as a reference when performing manual validation.

## Dockerfile Validation Checklist

### Base Image (Critical)
- [ ] No `:latest` tags used
- [ ] Specific version pinned (e.g., `node:20.11.0-alpine`)
- [ ] Uses minimal base image (alpine, slim, or distroless variants)
- [ ] Trusted registry used (docker.io, ghcr.io, gcr.io, quay.io)
- [ ] Base image is actively maintained and supported

### Multi-Stage Build (High Priority)
- [ ] At least 2 stages for production builds (build + runtime)
- [ ] All stages named with `AS` keyword
- [ ] Artifacts copied between stages using `COPY --from=`
- [ ] Final stage uses minimal base image
- [ ] Build tools NOT present in final stage
- [ ] Only necessary files in final image

### Security (Critical)
- [ ] `USER` directive present (non-root user specified)
- [ ] Last `USER` is NOT root or 0
- [ ] No secrets in ENV variables
- [ ] No secrets in COPY/ADD commands
- [ ] No secrets in RUN commands
- [ ] No hardcoded passwords or API keys
- [ ] Uses `--chown` when copying files for non-root user

### Directory & Paths (High Priority)
- [ ] `WORKDIR` uses absolute paths (not relative)
- [ ] No use of `cd` in RUN commands
- [ ] `.dockerignore` file present
- [ ] Build context is minimal (excludes unnecessary files)

### Layer Optimization (Medium Priority)
- [ ] RUN commands combined where logical (using `&&`)
- [ ] Package manager cache cleaned (apt, apk, yum)
- [ ] Dependency files copied before source code (for better caching)
- [ ] Multi-line commands use backslash continuation
- [ ] No unnecessary layers created

### Package Management (High Priority)
- [ ] Package versions pinned (apt-get, apk, yum)
- [ ] Package cache cleaned after installation
- [ ] Uses `--no-install-recommends` for apt-get
- [ ] Uses `--virtual` for build dependencies in alpine
- [ ] Python: Versions pinned in requirements.txt
- [ ] Node: Uses `npm ci` instead of `npm install`

### Best Practices (Medium Priority)
- [ ] `HEALTHCHECK` instruction defined
- [ ] Uses `COPY` instead of `ADD` (unless extracting archives)
- [ ] `CMD` and `ENTRYPOINT` use JSON notation
- [ ] Labels included (maintainer, version, description)
- [ ] Appropriate EXPOSE directives for documented ports
- [ ] ENV variables clearly defined
- [ ] Build args used for customization points

### Documentation (Low Priority)
- [ ] Comments explain complex operations
- [ ] Each stage has descriptive comment
- [ ] Unusual patterns are documented
- [ ] Maintainer/author documented in labels

## Hadolint Rule Reference

| Rule | Description | Severity | Check |
|------|-------------|----------|-------|
| DL3000 | Use absolute WORKDIR | Error | [ ] |
| DL3001 | Some commands should not be used | Warning | [ ] |
| DL3002 | Last USER should not be root | Error | [ ] |
| DL3003 | Use WORKDIR to switch directories | Warning | [ ] |
| DL3006 | Always tag image explicitly | Warning | [ ] |
| DL3007 | Using latest is prone to errors | Warning | [ ] |
| DL3008 | Pin versions in apt-get install | Warning | [ ] |
| DL3009 | Delete apt-get lists after installing | Info | [ ] |
| DL3013 | Pin versions in pip | Warning | [ ] |
| DL3014 | Use -y flag with apt-get | Warning | [ ] |
| DL3015 | Avoid additional packages | Info | [ ] |
| DL3016 | Pin versions in npm | Warning | [ ] |
| DL3018 | Pin versions in apk add | Warning | [ ] |
| DL3019 | Use --no-cache with apk add | Info | [ ] |
| DL3020 | Use COPY instead of ADD | Error | [ ] |
| DL3022 | COPY --from should reference stage | Error | [ ] |
| DL3025 | Use JSON for CMD and ENTRYPOINT | Warning | [ ] |

## Docker Compose Validation Checklist

### Modern Syntax (Critical)
- [ ] NO `version` field present (obsolete since Compose v2.27.0)
- [ ] File uses modern Compose Specification syntax
- [ ] Compatible with Docker Compose v2.27.0+

### Service Configuration (High Priority)
- [ ] All services have specific image tags (no `:latest`)
- [ ] Restart policies defined (`restart: unless-stopped`)
- [ ] Service dependencies specified (`depends_on`)
- [ ] Container names specified (optional but recommended)
- [ ] All services have descriptive names (kebab-case)

### Health Checks (High Priority)
- [ ] Critical services have health checks
- [ ] Health check intervals are reasonable
- [ ] Health check timeout values appropriate
- [ ] Start period allows for initialization
- [ ] Retries configured for reliability

### Resource Management (Medium Priority)
- [ ] Resource limits defined (memory, CPU)
- [ ] Resource reservations specified
- [ ] PID limits set (if applicable)
- [ ] Ulimits configured (if needed)

### Networking (High Priority)
- [ ] Custom networks defined (not default bridge)
- [ ] Network drivers specified
- [ ] Network isolation implemented
- [ ] Port mappings use host IP when needed
- [ ] Internal networks used for backend services

### Volumes & Persistence (High Priority)
- [ ] Named volumes used for persistence
- [ ] Volume drivers specified
- [ ] Bind mounts use absolute paths
- [ ] Volume permissions appropriate
- [ ] No sensitive data in bind mounts
- [ ] Volumes section defines named volumes

### Environment & Configuration (Critical)
- [ ] Environment variables properly defined
- [ ] No hardcoded secrets in compose file
- [ ] `.env` file used for configuration
- [ ] Secrets managed with Docker secrets or external
- [ ] ENV files specified with `env_file`

### Security (Critical)
- [ ] No `privileged: true` (unless absolutely necessary)
- [ ] Capabilities properly configured
- [ ] User/group specified for services
- [ ] Read-only root filesystem (where applicable)
- [ ] Security options configured
- [ ] AppArmor/SELinux profiles defined

### Logging (Medium Priority)
- [ ] Logging drivers configured
- [ ] Log rotation configured
- [ ] Log size limits set
- [ ] Appropriate log drivers for environment

### Build Configuration (if using build)
- [ ] Build context clearly specified
- [ ] Build args defined if needed
- [ ] Dockerignore file present
- [ ] Target stage specified for multi-stage
- [ ] Cache from configured for faster builds

## Multi-Stage Build Specific Checks

### Stage Structure
- [ ] Build stage uses full toolchain image
- [ ] Build stage installs ALL dependencies
- [ ] Build stage compiles/builds artifacts
- [ ] Runtime stage uses minimal image
- [ ] Runtime stage copies only artifacts
- [ ] Runtime stage installs only runtime deps

### Node.js Multi-Stage
- [ ] Build stage: `npm ci` (not `npm install`)
- [ ] Build stage: `npm run build`
- [ ] Build stage: `npm run test` (optional)
- [ ] Runtime stage: `npm ci --only=production`
- [ ] Runtime stage: Copies from builder `dist/build` folder
- [ ] Runtime stage: Uses alpine variant

### Python Multi-Stage
- [ ] Build stage: Compiles wheels
- [ ] Build stage: Installs build dependencies
- [ ] Build stage: Creates virtual environment
- [ ] Runtime stage: Installs pre-built wheels
- [ ] Runtime stage: Minimal Python image (alpine/slim)
- [ ] Runtime stage: No compiler or build tools

### Go Multi-Stage
- [ ] Build stage: Full Go image
- [ ] Build stage: CGO_ENABLED=0 for static binary
- [ ] Build stage: Builds binary
- [ ] Runtime stage: `scratch` or `distroless`
- [ ] Runtime stage: Only binary copied
- [ ] Runtime stage: Certificates copied if needed

## Security Checklist

### Image Security
- [ ] Base images scanned for vulnerabilities
- [ ] No images older than 6 months without review
- [ ] Images from trusted sources only
- [ ] Image digests used for critical deployments
- [ ] Regular security updates applied

### Runtime Security
- [ ] Runs as non-root user (UID > 0)
- [ ] Minimal capabilities granted
- [ ] No unnecessary privileges
- [ ] Read-only root filesystem where possible
- [ ] Temporary directories properly configured

### Network Security
- [ ] Only necessary ports exposed
- [ ] Internal networks for backend communication
- [ ] No host network mode (unless required)
- [ ] Firewall rules configured
- [ ] TLS/SSL for external communications

### Data Security
- [ ] No secrets in image layers
- [ ] Secrets managed via secrets management
- [ ] Sensitive volumes properly protected
- [ ] Encryption at rest configured
- [ ] Backup strategy for persistent data

### Access Control
- [ ] Least privilege principle applied
- [ ] Appropriate user permissions
- [ ] No root access to containers
- [ ] Host resources properly isolated
- [ ] Volume mounts restricted

## CI/CD Integration Checklist

### Pre-commit Validation
- [ ] Pre-commit hook validates Dockerfiles
- [ ] Pre-commit hook checks Compose files
- [ ] Hook checks for obsolete version field
- [ ] Hook runs Hadolint/DCLint
- [ ] Hook prevents commits with critical issues

### Pipeline Validation
- [ ] Hadolint runs on all Dockerfiles
- [ ] Docker Compose config validation
- [ ] DCLint runs on Compose files
- [ ] Security scanning integrated (Trivy, Snyk)
- [ ] Build tests before merge
- [ ] Automated PR comments with issues

### Deployment Validation
- [ ] Validation runs before deployment
- [ ] Rollback on validation failure
- [ ] Health checks verified post-deployment
- [ ] Smoke tests after deployment
- [ ] Monitoring alerts configured

## Common Issue Quick Reference

### ❌ Using :latest Tag
**Check:** `grep -r ":latest" Dockerfile docker-compose.yml`
**Fix:** Pin specific version (e.g., `nginx:1.24-alpine`)

### ❌ Obsolete Version Field
**Check:** `grep "^version:" docker-compose.yml`
**Fix:** Remove the entire `version:` line

### ❌ Running as Root
**Check:** `grep "^USER" Dockerfile | tail -1`
**Fix:** Add `USER <non-root-user>` before CMD/ENTRYPOINT

### ❌ No Multi-Stage Build
**Check:** `grep -c "^FROM" Dockerfile`
**Fix:** Implement at least 2 stages

### ❌ Package Cache Not Cleaned
**Check:** Look for apt/apk without cleanup
**Fix:** Add `rm -rf /var/lib/apt/lists/*` or `--no-cache`

### ❌ Using ADD Instead of COPY
**Check:** `grep "^ADD" Dockerfile`
**Fix:** Replace with `COPY` unless extracting archives

### ❌ No Health Check
**Check:** `grep "HEALTHCHECK" Dockerfile`
**Fix:** Add appropriate HEALTHCHECK instruction

### ❌ No Restart Policy
**Check:** `grep "restart:" docker-compose.yml`
**Fix:** Add `restart: unless-stopped` to services

## Validation Priority Matrix

| Priority | Dockerfile | Docker Compose | Multi-Stage | Security |
|----------|-----------|----------------|-------------|----------|
| CRITICAL | No :latest, USER set | No version field | N/A | Non-root user |
| HIGH | HEALTHCHECK, WORKDIR | Restart policy, Health checks | 2+ stages | No secrets |
| MEDIUM | Layer optimization | Resource limits | Named stages | Minimal image |
| LOW | Comments, Labels | Logging config | Optimization | Hardening |

## Tool-Specific Validation

### Hadolint Configuration (.hadolint.yaml)
```yaml
ignored:
  - DL3008  # Pin apt versions (too strict for some use cases)
override:
  error:
    - DL3002  # Last USER must not be root
    - DL3020  # Use COPY not ADD
trustedRegistries:
  - docker.io
  - ghcr.io
  - gcr.io
```

### DCLint Configuration (.dclintrc.json)
```json
{
  "rules": {
    "no-version-field": "error",
    "require-quotes": "warning",
    "no-duplicate-keys": "error"
  }
}
```

## Quick Validation Commands

```bash
# Validate Dockerfile
hadolint Dockerfile

# Validate Docker Compose
docker compose config --quiet

# Check for obsolete version field
grep "^version:" docker-compose.yml

# Count FROM statements (multi-stage check)
grep -c "^FROM" Dockerfile

# Check final USER
grep "^USER" Dockerfile | tail -1

# Find :latest tags
grep -r ":latest" .

# Full validation
./validate-docker.sh
```

---

**Use this checklist to ensure comprehensive validation of all Docker configurations.**
