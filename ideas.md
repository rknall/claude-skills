# GitLab Stack Management Skills - Ideas

## Project Overview

This is a project template system for managing stacks of Docker containers with the following principles:

* Everything configured through docker-compose.yml and ./config directory
* Secrets stored in ./secrets and docker secrets
* docker-entrypoint.sh scripts only when containers don't support native secrets
* All container files owned by the user running docker (no root-owned files)
* _temporary directory for transient setup files (cleaned up after use)
* Setup script for environment preparation (packages, gitlab, claude, etc.)

## Core Stack Management Skills

### 1. stack-validator ⭐ PRIORITY
**Purpose**: Validates the entire stack structure before deployment

**Features**:
- Validates docker-compose.yml syntax and configuration
- Verifies ./config directory structure
- Ensures ./secrets are properly configured and not exposed
- Validates file ownership (no root-owned files)
- Checks for proper docker-entrypoint.sh scripts where needed
- Pre-flight validation before stack deployment
- Ensures _temporary directory is properly configured

**Why Priority**: Prevents issues before they happen, establishes quality gates

**Enhancement Ideas**:
1. **Quick-check Mode**: Fast validation focusing only on critical issues (< 5 seconds)
   - Skip non-critical checks
   - Only validate must-pass criteria
   - Useful for rapid iteration during development

2. **Pre-commit Hook Template**: Auto-validate before git commits
   - Generate .git/hooks/pre-commit script
   - Run quick validation automatically
   - Block commits if critical issues found
   - Provide skip option for emergencies

3. **CI/CD Templates**: Ready-to-use pipeline configurations
   - GitLab CI .gitlab-ci.yml template with validation stage
   - GitHub Actions workflow template
   - Pre-configured validation jobs
   - Badge generation for README

4. **Fix Suggestions Script**: Generate shell commands for common fixes (without executing)
   - Output executable shell script with fixes
   - User can review before running
   - Grouped by category (ownership, permissions, gitignore, etc.)
   - Dry-run mode to see what would be fixed

5. **Validation Report Templates**: HTML/PDF report generation
   - Professional HTML reports for audits
   - PDF export for documentation
   - Include graphs and statistics
   - Trend analysis over time

6. **Comparison Validator**: Compare stack against reference/template
   - Validate against "golden" reference stack
   - Identify deviations from template
   - Useful for maintaining consistency across multiple stacks
   - Support for organization-wide standards

7. **Migration Checker**: Validate when migrating from old patterns to new
   - Check compatibility with new stack patterns
   - Identify breaking changes
   - Provide migration roadmap
   - Validate incremental migration steps

8. **Health Check Scripts**: Generate pre-deployment health check scripts
   - Service dependency validation
   - Network connectivity checks
   - Resource availability validation
   - Port conflict detection

9. **Security Scanning Enhancements**:
   - CVE scanning for base images
   - Exposed port security audit
   - SSL/TLS configuration validation
   - Detect sensitive data in environment variables
   - Check for latest security patches

10. **Performance Validation**:
    - Resource limits validation (memory, CPU)
    - Health check configuration
    - Restart policy validation
    - Volume mount efficiency checks

---

### 2. stack-creator ⭐ PRIORITY
**Purpose**: Creates new stack projects from templates

**Features**:
- Sets up standard directory structure (./config, ./secrets, ./_temporary)
- Generates initial docker-compose.yml with best practices
- Creates placeholder docker-entrypoint.sh scripts when needed
- Sets proper file permissions and ownership
- Initializes git repository with proper .gitignore for secrets
- Creates README template

**Why Priority**: Ensures consistency across all new stacks

---

### 3. secrets-manager ⭐ PRIORITY
**Purpose**: Manages docker secrets and ./secrets directory

**Features**:
- Manages docker secrets and ./secrets directory
- Helps migrate from environment variables to docker secrets
- Validates secret references in docker-compose.yml
- Ensures secrets aren't accidentally committed
- Generates secure random secrets
- Audits secret usage across the stack
- Identifies when docker-entrypoint.sh is needed for secret injection

**Why Priority**: Handles the most sensitive and error-prone aspect

---

## Supporting Skills (Future Development)

### 4. stack-debugger
**Purpose**: Diagnoses common stack issues

**Features**:
- Checks container logs and health
- Validates network connectivity between containers
- Identifies permission issues
- Checks for orphaned volumes or containers
- Provides actionable troubleshooting steps
- Analyzes _temporary directory for leftover files

---

### 5. config-generator
**Purpose**: Generates service-specific configuration files

**Features**:
- Generates service-specific configuration files for ./config
- Creates configuration templates for common services (nginx, postgres, redis, etc.)
- Validates configuration file syntax
- Manages environment-specific configs (dev/staging/prod)
- Ensures configs follow stack patterns

---

### 6. cleanup-manager
**Purpose**: Manages cleanup operations

**Features**:
- Manages the ./_temporary directory
- Cleans up after operations
- Safely removes unused volumes, images, and containers
- Provides disk space analysis
- Automates periodic cleanup tasks
- Ensures no root-owned files remain

---

### 7. migration-helper
**Purpose**: Migrates existing setups to stack pattern

**Features**:
- Helps migrate existing Docker setups to stack pattern
- Converts docker run commands to docker-compose.yml
- Identifies and converts environment variables to secrets
- Fixes file ownership issues
- Generates required docker-entrypoint.sh scripts

---

### 8. documentation-generator
**Purpose**: Creates and maintains stack documentation

**Features**:
- Creates README.md for each stack
- Documents services, ports, and dependencies
- Generates setup instructions
- Creates architecture diagrams
- Maintains changelog of stack modifications
- Documents secret requirements

---

## Implementation Priority

**Phase 1** (Immediate): ✅ **COMPLETE**
1. ✅ stack-validator (v1.0.0)
2. ✅ stack-creator (v1.0.0)
3. ✅ secrets-manager (v1.0.0)
4. ✅ config-generator (v1.0.0) - includes docker-validation

**Phase 2** (Short-term):
4. stack-debugger
5. cleanup-manager

**Phase 3** (Medium-term):
6. migration-helper

**Phase 4** (Long-term):
8. documentation-generator

---

## Common Patterns Across All Skills

All skills should:
- Respect the file ownership principle (no root files)
- Use ./_temporary for any temporary files
- Validate ./config and ./secrets directories
- Check docker-compose.yml compatibility
- Provide clear, actionable feedback
- Support both interactive and CI/CD usage
- Clean up after themselves

---

## Integration Points

- All skills should work together seamlessly
- stack-validator should be callable by other skills
- secrets-manager should integrate with stack-creator
- cleanup-manager should be called after operations
- Common utilities should be shared across skills

---

## Phase 1 Completion Notes (2025-10-20)

**Status**: COMPLETE ✅

All Phase 1 priority skills have been implemented and are production-ready:

### stack-validator (v1.0.0)
- Validates entire stack structure before deployment
- Ensures proper architecture patterns
- Checks directory structure, secrets management, .env configuration
- Detects issues and provides actionable guidance

### stack-creator (v1.0.0)
- Creates new stack projects from scratch
- Integrates all Phase 1 skills seamlessly
- Sets up git with main branch and ff-only merges
- Generates validation scripts and hooks
- Creates comprehensive documentation
- Enforces complete validation before completion
- NEVER uses workarounds - always asks user for guidance

### secrets-manager (v1.0.0)
- Manages Docker secrets for GitLab stack projects
- Ensures secrets never in .env or docker-compose.yml
- Handles migration from environment variables
- Validates, audits, and generates secure secrets
- Creates docker-entrypoint.sh when needed

### config-generator (v1.0.0)
- Generates service-specific configuration files
- Creates nginx, PostgreSQL, Redis configs
- Uses .env as primary config source
- Generates meta files (CLAUDE.md, .gitignore, .dockerignore)
- Strict validation for secrets and paths
- Integrates docker-validation for all Docker configs

### Integration

All skills work together seamlessly:
- stack-creator uses stack-validator, secrets-manager, and config-generator
- config-generator uses docker-validation
- secrets-manager integrates with stack-validator
- Complete validation enforced at every step

### Key Achievements

- ✅ 4 production-ready skills
- ✅ Complete integration between skills
- ✅ No-workaround policy enforced
- ✅ Comprehensive documentation (3,260+ lines for stack-creator alone)
- ✅ Git hooks and validation scripts
- ✅ Templates for common stacks
- ✅ Architecture decision records
- ✅ Marketplace version: 0.7.1

**Phase 1 is complete and ready for production use!**

---

*Document created: October 20, 2025*
*Last updated: October 20, 2025 - Phase 1 Complete*
