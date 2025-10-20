# Newt Blueprint Generator

Generate and validate Pangolin Newt blueprint configurations in YAML or Docker Labels format.

## Overview

This skill provides expert assistance for creating, validating, and managing Pangolin Newt blueprint configurations. It supports both YAML configuration files and Docker Compose label-based configurations.

## When to Use

Use this skill when you need to:
- Create Pangolin blueprint configurations
- Generate YAML configuration files for Newt
- Create Docker Compose files with Pangolin labels
- Configure proxy resources (HTTP, TCP, UDP)
- Set up client resources for Pangolin Olm
- Configure authentication (SSO, basic auth, pincode, password)
- Validate blueprint configurations
- Troubleshoot blueprint validation errors
- Convert between YAML and Docker Labels formats

## Installation

```bash
/plugin install newt-blueprint-generator
```

## Usage

### Triggering the Skill

The skill automatically activates when you mention:
- "Newt blueprint"
- "Pangolin blueprint"
- "Generate blueprint configuration"
- "Create proxy resource"
- "Pangolin YAML config"
- "Docker labels for Pangolin"

### Example Prompts

1. **Simple Web Application**:
   ```
   Create a Newt blueprint for a web app running on localhost:8080
   accessible at app.example.com
   ```

2. **TCP Database**:
   ```
   Generate a blueprint for exposing a PostgreSQL database on port 5432
   ```

3. **Docker Compose Setup**:
   ```
   Create a Docker Compose file with Pangolin labels for an nginx
   service at nginx.example.com
   ```

4. **Secured Resource**:
   ```
   Generate a blueprint with SSO authentication for secure.example.com
   ```

5. **Multi-Target Resource**:
   ```
   Create a blueprint with multiple backend targets for load balancing
   ```

## Features

### Comprehensive Blueprint Generation

- **HTTP Proxy Resources**: Full domain-based routing with headers, rules, and authentication
- **TCP/UDP Proxy Resources**: Raw port-based proxying for databases, game servers, etc.
- **Client Resources**: Olm client resources for SSH, RDP, and other protocols
- **Authentication**: SSO, basic auth, pincode, and password authentication
- **Access Control**: IP, CIDR, path, and country-based rules

### Validation

The skill automatically validates:
- Protocol-specific requirements (HTTP vs TCP/UDP)
- Unique constraints (`full-domain`, `proxy-port`)
- Authentication compatibility (HTTP only)
- Target method requirements
- Port ranges (1-65535)
- Email format for SSO users

### Format Support

- **YAML Configuration**: Standalone files for Newt CLI or API
- **Docker Labels**: Embedded in Docker Compose files

## Configuration Examples

### HTTP Proxy Resource (YAML)

```yaml
proxy-resources:
  web-app:
    name: Web Application
    protocol: http
    full-domain: app.example.com
    headers:
      - name: X-Custom-Header
        value: custom-value
    targets:
      - hostname: localhost
        port: 8080
        method: https
```

### TCP Proxy Resource (YAML)

```yaml
proxy-resources:
  database:
    name: PostgreSQL Database
    protocol: tcp
    proxy-port: 5432
    targets:
      - hostname: localhost
        port: 5432
```

### Docker Compose with Labels

```yaml
services:
  nginx:
    image: nginx:latest
    labels:
      - pangolin.proxy-resources.web.name=Web Server
      - pangolin.proxy-resources.web.full-domain=web.example.com
      - pangolin.proxy-resources.web.protocol=http
      - pangolin.proxy-resources.web.targets[0].method=http
```

## Common Validation Errors

The skill helps you resolve common errors:

| Error | Cause | Solution |
|-------|-------|----------|
| Duplicate 'full-domain' | Same domain used twice | Use unique subdomains |
| Duplicate 'proxy-port' | Same port used twice | Assign unique ports |
| Missing 'method' field | HTTP target without method | Add `method: http/https/h2c` |
| Auth on TCP/UDP | Auth not supported | Remove auth or use HTTP |
| Admin in sso-roles | Reserved role | Remove "Admin" from roles |

## Best Practices

1. **Use Descriptive IDs**: Name resources clearly (e.g., `web-app-prod`, `db-backup`)
2. **Enable Authentication**: Secure HTTP resources with SSO or password
3. **Document Ports**: Keep track of port assignments to avoid conflicts
4. **Explicit Sites**: Specify `site` for multi-site deployments
5. **Path Matching**: Use `prefix` for broad matches, `exact` for specific endpoints
6. **Validate Before Deploy**: Test configurations locally before production

## Version History

- **v1.0.0** (2025-01-20): Initial release with full Pangolin blueprint support

## Resources

- [Pangolin Documentation](https://docs.pangolin.net/manage/blueprints)
- [API Reference](https://api.pangolin.net/v1/docs/#/Organization/put_org__orgId__blueprint)
- [Example Python Script](https://github.com/fosrl/pangolin/blob/dev/blueprint.py)

## Support

For issues or questions:
- Pangolin GitHub: https://github.com/fosrl/pangolin
- Pangolin Slack: https://pangolin.net/slack
- Pangolin Discord: https://pangolin.net/discord

## License

This skill follows the same license as the Skills marketplace repository.
