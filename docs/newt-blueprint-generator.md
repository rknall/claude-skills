# Newt Blueprint Generator Skill

**Version:** 1.0.0
**Created:** 2025-01-20
**Category:** Development

## Overview

The Newt Blueprint Generator skill provides comprehensive assistance for creating and validating Pangolin Newt blueprint configurations. It supports both YAML configuration files and Docker Compose label-based configurations.

## Creation Process

This skill was created by:
1. Fetching documentation from https://docs.pangolin.net/manage/blueprints
2. Extracting comprehensive information about blueprint structure, validation rules, and examples
3. Creating a structured skill with detailed workflows and reference materials

## Files

### Core Files

- **SKILL.md**: Main skill definition with YAML frontmatter containing:
  - Complete overview of Pangolin blueprint formats
  - Resource types (proxy resources, client resources)
  - Authentication configuration patterns
  - Docker Labels format examples
  - Configuration properties reference
  - Validation rules and constraints
  - Common validation errors with solutions
  - Workflow for generating blueprints
  - Best practices
  - Example use cases

- **README.md**: User-facing documentation with:
  - Installation instructions
  - Usage examples
  - Feature overview
  - Configuration examples (YAML and Docker Compose)
  - Common validation errors reference
  - Best practices
  - Version history

- **validation-reference.md**: Comprehensive validation reference with:
  - Resource-level validations
  - Property constraints
  - Common validation errors with solutions
  - Valid/invalid configuration examples
  - Validation checklist

## Skill Capabilities

### Blueprint Generation

1. **YAML Configuration Files**
   - Standalone configuration files
   - API deployment support
   - Newt CLI integration

2. **Docker Labels**
   - Embedded in Docker Compose files
   - Automatic container discovery
   - Configuration merging across containers

### Resource Types

1. **Proxy Resources**
   - HTTP: Domain-based routing with headers, rules, auth
   - TCP/UDP: Port-based proxying for databases, game servers

2. **Client Resources**
   - Olm client resources for SSH, RDP, etc.
   - Port mapping configuration

### Authentication

- SSO (role-based and user-based)
- Basic authentication
- Pincode (6-digit)
- Password protection
- Whitelist users

### Access Control Rules

- IP-based rules
- CIDR-based rules
- Path-based rules
- Country-based rules
- Actions: allow, deny, pass

### Validation

- Protocol-specific requirements
- Unique domain/port constraints
- Authentication compatibility
- Target method requirements
- Port range validation
- Email format validation
- String length constraints

## Skill Triggers

The skill activates when users mention:
- "Newt blueprint"
- "Pangolin blueprint"
- "Generate blueprint configuration"
- "Create proxy resource"
- "Pangolin YAML config"
- "Docker labels for Pangolin"

## Example Use Cases

1. **Simple Web Application**
   - Expose web app via HTTPS
   - Domain-based routing

2. **TCP Database Access**
   - Expose PostgreSQL database
   - Port-based access

3. **Multi-Target Load Balancing**
   - Multiple backend servers
   - Same domain, different targets

4. **Secured Resource with SSO**
   - Web app with authentication
   - Role-based access control

## Validation Capabilities

The skill validates:
- Protocol-specific requirements (HTTP vs TCP/UDP)
- Unique constraints (full-domain, proxy-port)
- Authentication compatibility (HTTP only)
- Target method requirements
- Port ranges (1-65535)
- Email format for SSO users
- Pincode format (exactly 6 digits)
- Basic auth completeness
- String length constraints

## Best Practices Included

1. Use descriptive, kebab-case resource IDs
2. Enable authentication for sensitive HTTP resources
3. Document port assignments
4. Explicitly specify site for multi-site deployments
5. Use appropriate path matching types
6. Add custom headers for backend requirements
7. Order rules from specific to general
8. Validate before deployment
9. Include comments in configurations
10. Follow security recommendations

## Resources Referenced

- Pangolin Documentation: https://docs.pangolin.net/manage/blueprints
- API Documentation: https://api.pangolin.net/v1/docs/#/Organization/put_org__orgId__blueprint
- Example Python Script: https://github.com/fosrl/pangolin/blob/dev/blueprint.py

## Version History

### v1.0.0 (2025-01-20)
- Initial release
- Full support for YAML and Docker Labels formats
- Comprehensive validation
- HTTP, TCP, UDP proxy resources
- Client resources for Olm
- Authentication (SSO, basic auth, pincode, password)
- Access control rules
- Detailed error messages
- Best practices and examples
- Validation reference guide

## Future Enhancements

Potential future additions:
- Interactive blueprint builder workflow
- Blueprint templates for common scenarios
- Migration guide from manual configs
- Integration with Pangolin API for deployment
- Blueprint diffing and comparison
- Configuration testing utilities
- Health check configuration
- Advanced routing patterns

## Notes

- This skill was generated from official Pangolin documentation
- All validation rules are based on current (2025) Pangolin requirements
- The skill focuses on Newt (site agent) blueprint configurations
- Docker socket access required for Docker Labels format
- API key required for API-based deployment
