# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository Purpose

This is a **Claude Code Skills Marketplace** containing custom skills that extend Claude's capabilities. Skills are specialized prompts with supporting documentation that guide Claude to perform expert-level tasks in specific domains.

## Repository Structure

```
Skills/
├── .claude-plugin/
│   └── marketplace.json          # Marketplace configuration with all skills metadata
├── {skill-name}/                 # Each skill has its own directory
│   ├── SKILL.md                  # Main skill definition with YAML frontmatter (REQUIRED)
│   ├── README.md                 # User-facing documentation (REQUIRED)
│   └── *.md                      # Supporting resources (checklists, patterns, templates)
└── CLAUDE.md                     # This file
```

## Skill Categories

Current skills are organized into two categories:

**Development**: Architecture and design reviews
- `python-architecture-review`: Backend Python application architecture reviews
- `ui-design-review`: UI/UX and WCAG 2.1/2.2 accessibility compliance reviews

**Design**: Creative generation and design work
- `web-design-builder`: HTML5/JavaScript web design generation with Playwright verification
- `svg-logo-designer`: Professional SVG logo creation with multiple variations

## Working with Skills

### Skill File Structure

Every skill MUST have this structure:

**SKILL.md** (Required):
```markdown
---
name: "Display Name"
description: "What the skill does and WHEN to use it. Be explicit about trigger phrases."
---

# Skill Name

[Comprehensive instructions for Claude...]
```

**README.md** (Required):
- User-facing documentation
- Installation instructions
- Usage examples
- When to use the skill

**Supporting Files** (Optional but recommended):
- Checklists (e.g., `wcag-checklist.md`, `architecture-checklist.md`)
- Pattern libraries (e.g., `common-patterns.md`, `design-patterns-library.md`)
- Reference materials (e.g., `technology-recommendations.md`, `design-templates.md`)

### Skill Design Principles

1. **Comprehensive Instructions**: Skills should contain detailed, step-by-step workflows that Claude can follow autonomously
2. **Clear Triggers**: The `description` field must explicitly state when Claude should use this skill
3. **Supporting Resources**: Complex skills should include reference materials that Claude can consult
4. **Structured Output**: Skills should define clear output formats and deliverables
5. **Best Practices**: Include domain-specific best practices and current (2025) recommendations

### Creating New Skills

When adding a new skill:

1. **Create skill directory**: `mkdir new-skill-name`
2. **Create SKILL.md** with YAML frontmatter and comprehensive instructions
3. **Create README.md** with user documentation
4. **Add supporting resources** as separate `.md` files
5. **Update marketplace.json**:
   ```json
   {
     "name": "skill-name",
     "source": "./skill-name",
     "description": "Clear description with trigger phrases",
     "version": "1.0.0",
     "category": "development" | "design"
   }
   ```
6. **Update main README.md** with new skill documentation
7. **Increment marketplace version** in `marketplace.json`

### Modifying Existing Skills

When updating a skill:

1. **Edit skill files** (SKILL.md, README.md, supporting docs)
2. **Update version number** in `marketplace.json`
3. **Update main README.md** if significant changes
4. **Document changes** in README.md version history
5. **Test locally** before committing

## Marketplace Configuration

The `.claude-plugin/marketplace.json` file is the source of truth for:
- Marketplace metadata (name, version, description)
- All available skills and their metadata
- Skill categorization

**Current marketplace version**: Check `marketplace.json` for latest version

## Testing Skills Locally

Before committing skill changes:

1. Install the skill locally:
   ```bash
   /plugin install skill-name
   ```

2. Test by triggering the skill with appropriate prompts

3. Verify all supporting resources are accessible

4. Check that output matches expected format

## Skill Architecture Patterns

### Review Skills (Python Architecture Review, UI/UX Design Review)

Structure:
- **When to Use This Skill**: Clear trigger conditions
- **Review Framework**: Comprehensive evaluation dimensions
- **Review Output Format**: Structured deliverables
- **Communication Style**: How to interact with users
- **Reference Standards**: Industry standards and guidelines
- **Tools & Resources**: Recommended tools for the domain

### Generation Skills (Web Design Builder, SVG Logo Designer)

Structure:
- **When to Use This Skill**: Clear trigger conditions
- **Core Workflow**: Phase-by-phase process
- **Requirements Gathering**: Questions to ask users
- **Generation/Creation**: Step-by-step creation process
- **Verification/Review**: Quality checks and testing
- **Deliverables**: What to provide to users
- **Best Practices**: Domain-specific patterns and examples

## Git Workflow

This repository uses standard git practices:

**Branch**: `main` (primary branch)

**Commit conventions**: Use descriptive commit messages that explain what was added/changed

**Testing**: Always test skills locally before pushing

## Key Files to Update When Making Changes

| Change Type | Files to Update |
|-------------|----------------|
| New skill | `marketplace.json`, `README.md`, new skill directory |
| Skill update | Skill files, `marketplace.json` (version), potentially `README.md` |
| Marketplace metadata | `marketplace.json`, `README.md` |
| Documentation | Relevant `README.md` and skill documentation files |

## Important Notes

- **YAML Frontmatter**: SKILL.md must have valid YAML frontmatter with `name` and `description`
- **Descriptions**: Should explain BOTH what the skill does AND when to use it
- **Supporting Files**: Keep supporting documentation in separate `.md` files for modularity
- **Versioning**: Follow semantic versioning (MAJOR.MINOR.PATCH)
- **Categories**: Currently using "development" and "design" - maintain consistency
- **File Naming**: Use kebab-case for directories and files (e.g., `ui-design-review`)

## Distribution

This marketplace is designed to be distributed via git repository and installed with:
```bash
/plugin marketplace add rknall/Skills
```

Skills can then be installed individually or as a set.
