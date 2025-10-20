# rknall Custom Claude Code Skills

This repository contains custom skills for Claude Code, designed to enhance development workflows with specialized expertise.

## About This Marketplace

This is a personal skills marketplace that provides production-ready skills for various development tasks. All skills are designed to work seamlessly with Claude Code and can be installed automatically.

## Installation

### Add This Marketplace

If this repository is hosted on GitHub (e.g., `github.com/rknall/Skills`), add it to Claude Code:

```bash
/plugin marketplace add rknall/Skills
```

For other git hosting:
```bash
/plugin marketplace add <git-repository-url>
```

### Install Individual Skills

Once the marketplace is added, you can browse and install skills:

```bash
/plugin install python-architecture-review
```

Or install all skills from this marketplace at once through the plugin menu.

## Available Skills

### 1. Python Backend Architecture Review

**Version:** 1.0.0
**Category:** Development
**Description:** Comprehensive design architecture review for Python backend applications

A production-ready skill that provides expert-level architecture reviews covering:
- System architecture and design patterns
- Database design and optimization
- API design (REST/GraphQL/gRPC)
- Security architecture
- Scalability and performance
- Observability and monitoring
- Deployment strategies
- Code organization
- Resilience patterns

**Key Features:**
- Comprehensive review framework across 10+ dimensions
- Python-specific recommendations and best practices
- 12+ reference implementations of common patterns
- Technology stack recommendations for 2025
- Detailed architecture checklist
- Structured review outputs with prioritized recommendations

**When to Use:**
- Reviewing backend architecture designs
- Technology stack selection
- Scalability planning
- Security assessments
- Performance optimization planning

[View Full Documentation](./python-architecture-review/README.md)

### 2. UI/UX Design Review

**Version:** 1.0.0
**Category:** Development
**Description:** Comprehensive UI/UX design review with extensive accessibility analysis

A production-ready skill that provides expert-level design reviews covering:
- **Accessibility**: Complete WCAG 2.1/2.2 compliance checking (Level A, AA, AAA)
- **Visual Design**: Layout, typography, color theory, hierarchy, consistency
- **User Experience**: Usability heuristics, user flows, interaction patterns
- **Responsive Design**: Mobile-first approach, breakpoint strategy, adaptation
- **Components**: Button states, forms, modals, navigation, error handling
- **Desktop Applications**: Platform-specific patterns for Windows, macOS, Linux

**Key Features:**
- Comprehensive WCAG 2.1/2.2 compliance checklist
- Accessible component pattern library with code examples
- Testing tools and methodologies guide
- Platform-specific guidelines (Web, Windows, macOS, Linux)
- Color contrast analysis and recommendations
- Screen reader compatibility testing
- Keyboard navigation evaluation
- Prioritized recommendations (Critical → Low)

**When to Use:**
- Accessibility audits (WCAG compliance)
- Design system reviews
- Website/app usability evaluation
- Responsive design assessment
- Component library reviews
- Desktop application UI reviews
- Color palette and contrast verification

[View Full Documentation](./ui-design-review/README.md)

### 3. Docker Configuration Validator

**Version:** 1.0.0
**Category:** Development
**Description:** Comprehensive Docker and Docker Compose validation following best practices and security standards

A production-ready skill that provides expert-level Docker configuration validation:
- **Dockerfile Validation**: Syntax, multi-stage builds, security, best practices
- **Docker Compose Validation**: Modern syntax (no obsolete version field), service configuration
- **Security Audit**: Non-root users, exposed secrets, vulnerable base images
- **Multi-Stage Build Verification**: Proper stage implementation and optimization
- **Automation**: Generates validation scripts, CI/CD integrations, pre-commit hooks
- **Comprehensive Reporting**: Detailed issues with fixes, prioritized by severity

**Key Features:**
- Hadolint integration for deep Dockerfile analysis
- DCLint integration for Compose file linting
- Modern Compose syntax enforcement (v2.27.0+)
- Multi-stage build pattern validation
- Security vulnerability identification
- Automated validation script generation
- GitHub Actions and GitLab CI templates
- Complete validation checklists

**When to Use:**
- Validate Dockerfiles before deployment
- Audit Docker security and compliance
- Modernize Docker Compose files
- Review production readiness
- Create automated validation workflows
- Set up CI/CD pipeline validation
- Fix Docker configuration issues

[View Full Documentation](./docker-validation/README.md)

### 4. Web Design Builder

**Version:** 1.0.0
**Category:** Design
**Description:** Create professional HTML5/JavaScript web designs with Playwright verification

A production-ready skill that generates complete, accessible web designs from specifications:
- **Design Generation**: HTML5, CSS3, JavaScript for any web interface
- **Playwright Integration**: Automatic verification when MCP is available
- **Accessibility Testing**: WCAG 2.1 Level AA compliance checking
- **Visual Verification**: Screenshots at mobile, tablet, desktop breakpoints
- **Functionality Testing**: Forms, interactions, JavaScript validation
- **Performance Analysis**: Load time, resource optimization
- **Framework Support**: Vanilla, Tailwind CSS, React, Vue, Alpine.js

**Key Features:**
- Automatic Playwright MCP detection and graceful fallback
- Responsive, mobile-first designs
- Complete accessibility compliance (WCAG AA)
- Interactive component testing
- Performance metrics and optimization
- Design templates for common patterns
- Comprehensive verification reports

**When to Use:**
- Create landing pages or web applications
- Build responsive web interfaces
- Generate design mockups
- Refactor existing HTML/CSS/JS
- Create accessible WCAG-compliant designs
- Build component libraries

[View Full Documentation](./web-design-builder/README.md)

### 5. SVG Logo Designer

**Version:** 1.0.0
**Category:** Design
**Description:** Generate professional SVG logos with multiple variations

A production-ready skill that creates scalable vector graphic logos:
- **Multiple Concepts**: 3-5 different design directions per request
- **Layout Variations**: Horizontal, vertical, square, icon-only, text-only
- **Logo Types**: Wordmarks, lettermarks, pictorial, abstract, combination, emblems
- **Color Variations**: Full color, monochrome dark/light, reversed
- **Professional SVG**: Clean, optimized, accessible code
- **Usage Guidelines**: Complete brand identity documentation
- **Export Instructions**: SVG to PNG conversion guidance

**Key Features:**
- Multiple logo concepts exploring different visual approaches
- Comprehensive layout options for all use cases
- Color psychology guidance and palette selection
- Scalable vector graphics work at any size
- Accessibility with title/desc elements
- Complete usage documentation
- File organization and naming conventions

**When to Use:**
- Create brand logos or visual identities
- Design icons or symbols
- Generate logo variations and concepts
- Produce scalable graphics for branding
- Create wordmarks or lettermarks
- Design for both digital and print applications

[View Full Documentation](./svg-logo-designer/README.md)

### 6. GitLab Stack Validator

**Version:** 1.0.0
**Category:** Development
**Description:** Validates GitLab stack projects before deployment with comprehensive checks

A production-ready skill that validates stack project configurations:
- **Directory Structure**: Required directories (./config, ./secrets, ./_temporary), permissions, .gitignore
- **Environment Variables**: .env and .env.example synchronization (critical requirement)
- **Docker Configuration**: Uses docker-validation skill for compose and Dockerfile checks
- **Secrets Management**: Docker secrets validation, permission checks, exposure detection
- **Configuration Files**: Syntax validation, proper organization, no embedded secrets
- **File Ownership**: Detects root-owned files, ensures proper permissions
- **Scripts**: docker-entrypoint.sh validation when necessary
- **Temporary Files**: ./_temporary directory usage and cleanup validation

**Key Features:**
- Critical .env/.env.example synchronization checking
- Integrates with docker-validation skill for Docker-specific validation
- Comprehensive security scanning for exposed secrets
- File ownership auditing (no root-owned files)
- Detailed, actionable validation reports
- Multiple output formats (text, JSON)
- Strict and permissive validation modes
- Custom validation rules support
- Works with companion skills (stack-creator, secrets-manager)

**When to Use:**
- Validate stack before deployment
- Pre-deployment health checks
- Audit stack configuration and security
- Verify .env and .env.example are in sync
- Check secrets management compliance
- Ensure Docker best practices
- CI/CD pipeline validation gates
- Identify configuration issues early

[View Full Documentation](./stack-validator/README.md)

### 7. GitLab Stack Secrets Manager

**Version:** 1.0.0
**Category:** Development
**Description:** Secure Docker secrets management - ensures secrets never in .env or docker-compose.yml

A production-ready skill that manages Docker secrets securely:
- **Secret Migration**: Moves secrets from .env and docker-compose.yml to Docker secrets
- **Secret Creation**: Generates secure random secrets with proper permissions
- **Validation**: Detects secrets in wrong locations (critical security issue)
- **Auditing**: Find leaks, unused secrets, permission issues
- **Git Protection**: Ensures secrets never committed to version control
- **docker-entrypoint.sh**: Generates entrypoint scripts when containers lack native support

**Key Features:**
- Critical security focus: NO secrets in .env or docker-compose.yml environment
- Automatic migration from insecure locations
- Secure random secret generation (alphanumeric, hex, base64, UUID)
- Comprehensive leak detection across .env, compose, config files, git history
- File permission management (700/600)
- Integration with stack-validator for security checks
- Works with stack-creator for proper initialization

**When to Use:**
- Fix "secrets in .env" security issues
- Migrate environment variables to Docker secrets
- Create new secure secrets
- Validate secret configuration
- Audit secret usage and detect leaks
- Generate docker-entrypoint.sh for legacy containers
- Rotate existing secrets
- Ensure secrets not in git

[View Full Documentation](./secrets-manager/README.md)

### 8. GitLab Stack Config Generator

**Version:** 1.0.0
**Category:** Development
**Description:** Service configuration generator using .env as primary config source

A production-ready skill that generates service configurations:
- **Service Templates**: Nginx (3 variants), PostgreSQL (3 variants), Redis (3 variants)
- **Meta Files**: CLAUDE.md (with commit rules), .gitignore, .dockerignore
- **Configuration Source**: .env as single source of truth (not separate env files)
- **Directory Structure**: Service-specific directories (./config/service-name/)
- **Strict Validation**: No secrets in configs, .env/.env.example sync, path validation
- **Docker Integration**: Uses docker-validation skill for all Docker configs

**Key Features:**
- User-selectable template defaults (production, development, custom)
- Flat config structure inside each service directory
- Environment variable placeholders in all configs
- Critical .env and .env.example synchronization
- Secret detection with secrets-manager integration
- Path validation (all referenced paths must exist)
- Syntax validation per service type
- Meta files with proper git exclusions

**When to Use:**
- Generate service configurations (nginx, PostgreSQL, Redis)
- Set up project meta files (CLAUDE.md, .gitignore, .dockerignore)
- Create config templates for new services
- Ensure configs use .env variables correctly
- Validate existing configurations
- Sync .env and .env.example

[View Full Documentation](./config-generator/README.md)

### 9. Newt Blueprint Generator

**Version:** 1.0.0
**Category:** Development
**Description:** Generate and validate Pangolin Newt blueprint configurations

A production-ready skill that creates Pangolin Newt blueprints:
- **Blueprint Formats**: YAML configuration files and Docker Compose labels
- **Proxy Resources**: HTTP (domain-based), TCP/UDP (port-based) resource configurations
- **Client Resources**: Olm client resources for SSH, RDP, and other protocols
- **Authentication**: SSO, basic auth, pincode, and password authentication
- **Access Control**: IP, CIDR, path, and country-based rules
- **Validation**: Comprehensive validation with helpful error messages

**Key Features:**
- Support for both YAML and Docker Labels format
- Protocol-specific validation (HTTP vs TCP/UDP requirements)
- Authentication configuration with SSO role/user management
- Multi-target load balancing support
- Path-based routing with prefix/exact/regex matching
- Custom header injection
- Targets-only resource configuration for simplified setups
- Detailed validation error explanations
- Best practices and security recommendations

**When to Use:**
- Create Pangolin blueprint configurations
- Expose web applications via domain names (HTTP)
- Expose databases or other services via ports (TCP/UDP)
- Configure Olm client resources
- Set up authentication and access control
- Validate existing blueprint configurations
- Convert between YAML and Docker Labels formats
- Troubleshoot blueprint validation errors

[View Full Documentation](./newt-blueprint-generator/README.md)

### 10. GitLab Stack Creator

**Version:** 1.0.0
**Category:** Development
**Description:** Create new GitLab stack projects with complete validation

A production-ready skill that creates GitLab stack projects from scratch:
- **Directory Structure**: Proper ./config, ./secrets, ./_temporary, ./scripts, ./docs setup
- **Git Configuration**: Initializes repository with main branch and ff-only merge strategy
- **Validation Scripts**: Creates validate-stack.sh, pre-commit hooks, setup-hooks.sh
- **Docker Configuration**: Generates docker-compose.yml validated by docker-validation skill
- **Secrets Management**: Integrates secrets-manager for secure secret handling
- **Service Configs**: Uses config-generator for nginx, PostgreSQL, Redis configurations
- **Documentation**: Generates README.md, CLAUDE.md, setup.md, services.md, ADRs

**Key Features:**
- Integrates stack-validator, secrets-manager, docker-validation, config-generator skills
- Never uses workarounds - always asks user for guidance when stuck
- Complete only when ALL validators pass with NO issues
- Git hooks for pre-commit validation (blocks commits if validation fails)
- Comprehensive templates for common stacks (web, full-stack)
- Architecture decision records in ./docs/decisions/
- ff-only merge strategy for clean git history
- main as default branch name
- All scripts executable and ready to use

**When to Use:**
- Create new GitLab stack project from scratch
- Initialize Docker stack with proper structure
- Set up project with validation from the start
- Bootstrap production-ready stack following best practices
- Need git repository with validation hooks
- Want complete documentation generated automatically

**Completion Criteria:**
A stack is complete ONLY when:
- ✅ stack-validator reports NO issues
- ✅ secrets-manager is satisfied (NO open issues)
- ✅ docker-validation is satisfied (NO issues)
- ✅ All validation scripts execute successfully
- ✅ Git repository properly initialized and configured
- ✅ Documentation complete in ./docs

[View Full Documentation](./stack-creator/README.md)

## Repository Structure

```
Skills/
├── .claude-plugin/
│   └── marketplace.json               # Marketplace configuration
├── python-architecture-review/         # Python Backend Architecture Review Skill
│   ├── SKILL.md                       # Main skill definition
│   ├── README.md                      # Skill documentation
│   ├── architecture-checklist.md      # Complete review checklist
│   ├── common-patterns.md             # Reference implementations
│   └── technology-recommendations.md  # Tech stack guide
├── ui-design-review/                  # UI/UX Design Review Skill
│   ├── SKILL.md                       # Main skill definition
│   ├── README.md                      # Skill documentation
│   ├── wcag-checklist.md              # WCAG 2.1/2.2 compliance checklist
│   ├── design-patterns-library.md     # Accessible component patterns
│   └── testing-resources.md           # Testing tools and methods
├── docker-validation/                 # Docker Configuration Validator Skill
│   ├── SKILL.md                       # Main skill definition
│   ├── README.md                      # Skill documentation
│   ├── validation-checklist.md        # Complete validation checklist
│   └── tool-installation.md           # Tool setup guide
├── web-design-builder/                # Web Design Builder Skill
│   ├── SKILL.md                       # Main skill definition
│   ├── README.md                      # Skill documentation
│   └── design-templates.md            # Ready-to-use templates
├── svg-logo-designer/                 # SVG Logo Designer Skill
│   ├── SKILL.md                       # Main skill definition
│   └── README.md                      # Skill documentation
├── stack-validator/                   # GitLab Stack Validator Skill
│   ├── SKILL.md                       # Main skill definition
│   ├── README.md                      # Skill documentation
│   ├── validation-patterns.md         # Architecture patterns and examples
│   └── common-issues.md               # Issue reference guide
├── secrets-manager/                   # GitLab Stack Secrets Manager Skill
│   ├── SKILL.md                       # Main skill definition
│   ├── README.md                      # Skill documentation
│   ├── secrets-patterns.md            # Security patterns and best practices
│   └── migration-guide.md             # Step-by-step migration scenarios
├── config-generator/                  # GitLab Stack Config Generator Skill
│   ├── SKILL.md                       # Main skill definition
│   ├── README.md                      # Skill documentation
│   ├── service-templates.md           # Service templates (nginx, postgres, redis)
│   └── validation-rules.md            # Validation rules reference
├── newt-blueprint-generator/          # Newt Blueprint Generator Skill
│   ├── SKILL.md                       # Main skill definition
│   ├── README.md                      # Skill documentation
│   └── validation-reference.md        # Validation rules reference
├── stack-creator/                     # GitLab Stack Creator Skill
│   ├── SKILL.md                       # Main skill definition
│   ├── README.md                      # Skill documentation
│   ├── git-hooks-guide.md             # Git hooks and validation scripts
│   ├── templates-reference.md         # docker-compose and config templates
│   └── workflow-examples.md           # Example workflows
└── README.md                          # This file
```

## Creating Your Own Skills

### Quick Start

1. Create a new directory for your skill:
```bash
mkdir my-new-skill
cd my-new-skill
```

2. Create a `SKILL.md` file with YAML frontmatter:
```markdown
---
name: "My Skill Name"
description: "What this skill does and when to use it"
---

# My Skill Name

## Instructions

[Your skill instructions here...]
```

3. Add your skill to `.claude-plugin/marketplace.json`:
```json
{
  "plugins": [
    {
      "name": "my-new-skill",
      "source": "./my-new-skill",
      "description": "Description of my skill",
      "version": "1.0.0"
    }
  ]
}
```

4. Test your skill locally by installing it:
```bash
/plugin install my-new-skill
```

### Skill Best Practices

1. **Clear Description**: Make sure your description explains both what the skill does AND when Claude should use it
2. **Comprehensive Instructions**: Provide detailed, step-by-step guidance
3. **Examples**: Include example usage and code snippets where relevant
4. **Reference Resources**: Add supporting documents (checklists, patterns, etc.)
5. **Version Control**: Update version numbers when making changes

### Skill Components

A well-structured skill can include:
- `SKILL.md` - Main skill definition (required)
- `README.md` - User-facing documentation
- Supporting markdown files - Reference materials, checklists, patterns
- Scripts or templates - Reusable code or configuration
- Examples - Sample inputs/outputs

## Using Skills in Your Projects

### Repository-Level Configuration

To ensure your team uses the same skills, add a `.claude/config.json` to your project repository:

```json
{
  "marketplaces": [
    "rknall/Skills"
  ],
  "plugins": [
    "python-architecture-review"
  ]
}
```

When team members trust your repository folder, Claude Code will automatically install these marketplaces and plugins.

### Local Testing

Before publishing, test your skills locally:

1. Copy the skill directory to `~/.claude/skills/`
2. Restart Claude Code or run `/reload`
3. Trigger the skill by asking Claude to perform the relevant task

## Marketplace Configuration

The `.claude-plugin/marketplace.json` file defines this marketplace:

```json
{
  "$schema": "https://anthropic.com/claude-code/marketplace.schema.json",
  "name": "rknall-custom-skills",
  "version": "1.0.0",
  "description": "Custom Claude Code skills marketplace for advanced development workflows",
  "owner": {
    "name": "rknall"
  },
  "plugins": [...]
}
```

## Distribution

This marketplace can be distributed via:
- GitHub repository (recommended)
- GitLab repository
- Any git hosting service
- Direct URL to git repository

Users install it with a single command:
```bash
/plugin marketplace add rknall/Skills
```

## Version History

### 0.6.2 (2025-10-20)
- Added GitLab Stack Config Generator skill v1.0.0
  - Service-specific configuration generation for GitLab stack projects
  - .env as primary configuration source (single source of truth)
  - Service templates: Nginx (3 variants), PostgreSQL (3 variants), Redis (3 variants)
  - Meta files generation: CLAUDE.md (with commit message rules), .gitignore, .dockerignore
  - User-selectable template defaults (production, development, custom)
  - Strict .env and .env.example synchronization checking
  - Secret detection with secrets-manager integration
  - Path validation for all referenced files and directories
  - Docker validation using docker-validation skill (always)
  - Comprehensive service templates with complete examples

### 0.6.1 (2025-10-20)
- Added GitLab Stack Secrets Manager skill v1.0.0
  - Secure Docker secrets management for GitLab stack projects
  - Critical focus: ensures secrets NEVER in .env or docker-compose.yml
  - Automatic migration from insecure locations
  - Secret creation with secure random generation (alphanumeric, hex, base64, UUID)
  - Comprehensive validation and leak detection
  - docker-entrypoint.sh generation for legacy containers
  - File permission management (700/600)
  - Git protection and history scanning
  - Integration with stack-validator for security checks
  - Complete migration guide with step-by-step scenarios

### 0.6.0 (2025-10-20)
- Added GitLab Stack Validator skill v1.0.0
  - Comprehensive stack project validation before deployment
  - Critical .env and .env.example synchronization checking
  - Directory structure, secrets management, and ownership validation
  - Integrates with docker-validation skill for Docker-specific checks
  - Multiple validation modes (standard, strict, permissive)
  - JSON and text output formats for CI/CD integration
  - Works with companion skills: stack-creator and secrets-manager
  - Complete validation patterns and common issues reference guides

### 0.5.0 (2025-10-18)
- Added Docker Configuration Validator skill v1.0.0
  - Comprehensive Dockerfile and Docker Compose validation
  - Modern Compose syntax enforcement (no obsolete version field)
  - Multi-stage build verification
  - Security audit capabilities
  - Hadolint and DCLint integration
  - Automated validation script generation
  - CI/CD integration templates (GitHub Actions, GitLab CI)
  - Complete validation checklists and tool installation guides

### 0.4.0 (2025-10-18)
- Added Web Design Builder skill v1.0.0
  - HTML5/JavaScript design generation
  - Playwright MCP integration for automatic verification
  - Accessibility, visual, and functionality testing
  - Multiple framework support (Vanilla, Tailwind, React, Vue, Alpine.js)
  - Design templates for common patterns
- Added SVG Logo Designer skill v1.0.0
  - Professional logo generation with multiple concepts
  - Layout variations (horizontal, vertical, square, icon, text)
  - All major logo types supported
  - Color psychology guidance
  - Complete usage guidelines

### 0.2.0 (2025-10-18)
- Added UI/UX Design Review skill v1.0.0
  - Comprehensive WCAG 2.1/2.2 compliance framework
  - Accessible component pattern library
  - Visual design and UX evaluation
  - Testing tools and resources guide

### 0.1.0 (2025-10-18)
- Initial marketplace setup
- Python Backend Architecture Review skill v1.0.0
  - Comprehensive review framework
  - 12+ architectural patterns
  - Technology recommendations for 2025
  - Complete architecture checklist

## Contributing

To add or improve skills in this marketplace:

1. Create or modify the skill in its directory
2. Update the skill's README.md with documentation
3. Update marketplace.json with any new skills
4. Update version numbers appropriately
5. Test locally before committing

## Future Skills

Planned skills for future releases:
- Frontend Architecture Review (React/Vue/Angular)
- Infrastructure as Code Review (Terraform/CloudFormation)
- Kubernetes Configuration Validator (manifest validation, best practices)
- Security Audit Skill (penetration testing, threat modeling)
- Performance Optimization Skill (profiling, benchmarking)
- Database Schema Review Skill
- API Design Review Skill (REST, GraphQL, gRPC)
- Microservices Architecture Skill
- Code Quality Review Skill (refactoring, technical debt)
- Documentation Review Skill (technical writing, API docs)

## Support

For issues, questions, or suggestions:
- Open an issue in this repository
- Contact the maintainer directly

## Resources

- [Claude Code Documentation](https://docs.claude.com/en/docs/claude-code)
- [Skills Documentation](https://docs.claude.com/en/docs/claude-code/skills)
- [Plugin Marketplace Documentation](https://docs.claude.com/en/docs/claude-code/plugin-marketplaces)

## License

These skills are provided as-is for use with Claude Code. Individual skills may have their own licensing terms.

---

**Marketplace Version:** 0.6.2
**Last Updated:** 2025-10-20
**Maintainer:** rknall
