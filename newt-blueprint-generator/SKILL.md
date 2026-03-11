---
name: "newt-blueprint-generator"
description: "Generate and validate Pangolin Newt blueprint configurations in YAML or Docker Labels format. Use when creating Pangolin resource configurations, proxy resources, client resources, authentication settings, or Docker Compose blueprints."
---

# Newt Blueprint Generator

Expert assistance for creating, validating, and managing Pangolin Newt blueprint configurations.

## Overview

Pangolin Blueprints are declarative configurations that allow you to define resources and their settings in a structured format. They support two formats:

1. **YAML Configuration Files**: Standalone configuration files
2. **Docker Labels**: Configuration embedded in Docker Compose files

## Blueprint Formats

### YAML Configuration Format

YAML configs can be applied using:
- **Newt CLI**: Pass `--blueprint-file /path/to/blueprint.yaml`
- **API**: POST to `/org/{orgId}/blueprint` with base64-encoded JSON body

Example Newt usage:
```bash
newt --blueprint-file /path/to/blueprint.yaml <other-args>
```

### Docker Labels Format

For containerized applications, blueprints can be defined using Docker labels with the `pangolin.` prefix.

Enable Docker socket access:
```bash
newt --docker-socket /var/run/docker.sock <other-args>
```

Or use environment variable:
```bash
DOCKER_SOCKET=/var/run/docker.sock
```

## Resource Types

### Proxy Resources

Proxy resources expose HTTP, TCP, or UDP services through Pangolin.

#### HTTP Proxy Resource Example

```yaml
proxy-resources:
  resource-nice-id-uno:
    name: this is a http resource
    protocol: http
    full-domain: uno.example.com
    host-header: example.com
    tls-server-name: example.com
    headers:
      - name: X-Example-Header
        value: example-value
      - name: X-Another-Header
        value: another-value
    rules:
      - action: allow
        match: ip
        value: 1.1.1.1
      - action: deny
        match: cidr
        value: 2.2.2.2/32
      - action: pass
        match: path
        value: /admin
    targets:
      - site: lively-yosemite-toad
        hostname: localhost
        method: http
        port: 8000
      - site: slim-alpine-chipmunk
        hostname: localhost
        path: /admin
        path-match: exact
        method: https
        port: 8001
```

#### TCP/UDP Proxy Resource Example

```yaml
proxy-resources:
  resource-nice-id-dos:
    name: this is a raw resource
    protocol: tcp
    proxy-port: 3000
    targets:
      - site: lively-yosemite-toad
        hostname: localhost
        port: 3000
```

#### Targets-Only Resources

Simplified resources containing only target configurations:

```yaml
proxy-resources:
  additional-targets:
    targets:
      - site: another-site
        hostname: backend-server
        method: https
        port: 8443
      - site: another-site
        hostname: backup-server
        method: http
        port: 8080
```

**Note**: When using targets-only resources, `name` and `protocol` fields are not required.

### Client Resources

Client resources define proxied resources accessible via Olm client (SSH, RDP):

```yaml
client-resources:
  client-resource-nice-id-uno:
    name: this is my resource
    protocol: tcp
    proxy-port: 3001
    hostname: localhost
    internal-port: 3000
    site: lively-yosemite-toad
```

## Authentication Configuration

Authentication is **off by default**. Enable by adding fields in the `auth` section.

**Note**: Authentication is only allowed on HTTP resources, not TCP/UDP.

```yaml
proxy-resources:
  secure-resource:
    name: Secured Resource
    protocol: http
    full-domain: secure.example.com
    auth:
      pincode: 123456
      password: your-secure-password
      basic-auth:
        user: asdfa
        password: sadf
      sso-enabled: true
      sso-roles:
        - Member
        - Admin
      sso-users:
        - user@example.com
      whitelist-users:
        - admin@example.com
```

## Docker Labels Format

### Complete Docker Compose Example

```yaml
services:
  newt:
    image: fosrl/newt
    container_name: newt
    restart: unless-stopped
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
    environment:
      - PANGOLIN_ENDPOINT=https://app.pangolin.net
      - NEWT_ID=h1rbsgku89wf9z3
      - NEWT_SECRET=z7g54mbcwkglpx1aau9gb8mzcccoof2fdbs97keoakg2pp5z
      - DOCKER_SOCKET=/var/run/docker.sock

  nginx1:
    image: nginxdemos/hello
    container_name: nginx1
    labels:
      # Proxy Resource Configuration
      - pangolin.proxy-resources.nginx.name=nginx
      - pangolin.proxy-resources.nginx.full-domain=nginx.fosrl.io
      - pangolin.proxy-resources.nginx.protocol=http
      - pangolin.proxy-resources.nginx.headers[0].name=X-Example-Header
      - pangolin.proxy-resources.nginx.headers[0].value=example-value
      # Target Configuration - port and hostname auto-detected
      - pangolin.proxy-resources.nginx.targets[0].method=http
      - pangolin.proxy-resources.nginx.targets[0].path=/path
      - pangolin.proxy-resources.nginx.targets[0].path-match=prefix

  nginx2:
    image: nginxdemos/hello
    container_name: nginx2
    labels:
      # Additional target with explicit hostname and port
      - pangolin.proxy-resources.nginx.targets[1].method=http
      - pangolin.proxy-resources.nginx.targets[1].hostname=nginx2
      - pangolin.proxy-resources.nginx.targets[1].port=80

networks:
  default:
    name: pangolin_default
```

### Docker Labels Considerations

- **Automatic Discovery**: When hostname and internal port are not defined, Pangolin auto-detects from container configuration
- **Site Assignment**: If no site is specified, resource is assigned to the discovering Newt site
- **Configuration Merging**: Configuration across containers is merged to form complete resource definitions

## Configuration Reference

For complete property tables, validation rules, constraints, and common error solutions, see [validation-reference.md](./validation-reference.md).

Key constraints to remember:
- **HTTP resources** require `full-domain` (unique) and `method` on all targets
- **TCP/UDP resources** require `proxy-port` (unique); `method` and `auth` are NOT allowed
- **Targets-only resources** omit `name` and `protocol`
- **Authentication** is HTTP-only (SSO, basic auth, pincode, password)

## Workflow for Generating Blueprints

When a user requests a Pangolin Newt blueprint configuration:

1. **Gather Requirements**: Resource type (proxy/client), protocol, domain/port, targets, auth needs, format preference (YAML or Docker Labels).

2. **Select Format**: Use **YAML** for standalone/API deployment. Use **Docker Labels** for containerized applications.

3. **Generate Configuration**: Create well-structured YAML or Docker Compose with comments and kebab-case resource IDs.

4. **Validate Before Deployment**:
   - Lint the YAML syntax: `python -c "import yaml; yaml.safe_load(open('blueprint.yaml'))"`
   - Dry-run with Newt: `newt --blueprint-file blueprint.yaml --dry-run`
   - For Docker labels, verify syntax: `docker compose config`
   - Check the [validation-reference.md](./validation-reference.md) checklist for protocol-specific rules

5. **Error Recovery**: If validation fails:
   - Parse the error message (e.g., "Duplicate 'full-domain' values found")
   - Refer to the [Common Validation Errors](./validation-reference.md#common-validation-errors) section for fix patterns
   - Apply the fix, re-validate, and confirm resolution before deploying

6. **Deploy and Verify**:
   - Apply via CLI: `newt --blueprint-file blueprint.yaml <other-args>`
   - Or via API: `POST /org/{orgId}/blueprint` with base64-encoded JSON body
   - Confirm resources appear in Pangolin dashboard after deployment

## Best Practices

1. **Resource IDs**: Use descriptive, kebab-case identifiers (e.g., `web-app-prod`, `database-backup`)
2. **Target Organization**: Group related targets under the same resource ID
3. **Security First**: Enable authentication for sensitive HTTP resources
4. **Port Management**: Document port assignments to avoid conflicts
5. **Site Assignment**: Explicitly specify `site` for multi-site deployments
6. **Path Matching**: Use `prefix` for broad matches, `exact` for specific endpoints
7. **Headers**: Add custom headers for backend requirements (e.g., X-Forwarded-* headers)
8. **Rules**: Order rules from most specific to least specific
9. **Validation**: Always validate configurations before deployment
10. **Documentation**: Include comments in YAML or Docker Compose files explaining non-obvious choices

## Resources

- **Validation Reference**: [validation-reference.md](./validation-reference.md) — property tables, constraints, error solutions
- **README**: [README.md](./README.md) — installation and usage documentation
- **API Documentation**: https://api.pangolin.net/v1/docs/#/Organization/put_org__orgId__blueprint
- **Python Example**: https://github.com/fosrl/pangolin/blob/dev/blueprint.py
- **Official Docs**: https://docs.pangolin.net/manage/blueprints

## Example Use Cases

### Use Case 1: Simple Web Application

**Requirements**: Expose a web app running on localhost:8080 via HTTPS at app.example.com

```yaml
proxy-resources:
  web-app:
    name: Web Application
    protocol: http
    full-domain: app.example.com
    targets:
      - hostname: localhost
        port: 8080
        method: https
```

### Use Case 2: TCP Database Access

**Requirements**: Expose PostgreSQL database on port 5432

```yaml
proxy-resources:
  postgres-db:
    name: PostgreSQL Database
    protocol: tcp
    proxy-port: 5432
    targets:
      - hostname: localhost
        port: 5432
```

### Use Case 3: Multi-Target Load Balanced HTTP Service

**Requirements**: Multiple backend servers for the same domain

```yaml
proxy-resources:
  api-service:
    name: API Service
    protocol: http
    full-domain: api.example.com
    targets:
      - site: site-01
        hostname: backend-01
        port: 8080
        method: http
      - site: site-02
        hostname: backend-02
        port: 8080
        method: http
```

### Use Case 4: Secured Resource with SSO

**Requirements**: Web app with SSO authentication

```yaml
proxy-resources:
  secure-app:
    name: Secure Application
    protocol: http
    full-domain: secure.example.com
    auth:
      sso-enabled: true
      sso-roles:
        - Member
        - Developer
      sso-users:
        - admin@example.com
    targets:
      - hostname: localhost
        port: 3000
        method: https
```

