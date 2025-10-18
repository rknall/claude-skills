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

### Python Backend Architecture Review

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

## Repository Structure

```
Skills/
├── .claude-plugin/
│   └── marketplace.json          # Marketplace configuration
├── python-architecture-review/    # Python Backend Architecture Review Skill
│   ├── SKILL.md                  # Main skill definition
│   ├── README.md                 # Skill documentation
│   ├── architecture-checklist.md # Complete review checklist
│   ├── common-patterns.md        # Reference implementations
│   └── technology-recommendations.md # Tech stack guide
└── README.md                     # This file
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

### 1.0.0 (2025-10-18)
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
- Security Audit Skill
- Performance Optimization Skill
- Database Schema Review Skill
- API Design Review Skill
- Microservices Architecture Skill

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

**Marketplace Version:** 1.0.0
**Last Updated:** 2025-10-18
**Maintainer:** rknall
