# Validation Reference

This document provides a comprehensive reference for all validation rules and constraints in Pangolin Newt blueprints.

## Table of Contents

1. [Resource-Level Validations](#resource-level-validations)
2. [Property Constraints](#property-constraints)
3. [Common Validation Errors](#common-validation-errors)
4. [Validation Quick Reference](#validation-quick-reference)

## Resource-Level Validations

### Targets-Only Resources

A resource can contain **only** the `targets` field:

```yaml
proxy-resources:
  additional-targets:
    targets:
      - site: another-site
        hostname: backend-server
        method: https
        port: 8443
```

When using targets-only:
- `name` field is NOT required
- `protocol` field is NOT required
- All other resource-level validations are skipped

### Protocol-Specific Requirements

#### HTTP Protocol

**Required fields:**
- `full-domain` - Must be unique across all proxy resources
- All targets must have `method` field (`http`, `https`, or `h2c`)

**Optional features:**
- `auth` configuration (SSO, basic auth, pincode, password)
- `headers` array
- `rules` array
- `host-header`
- `tls-server-name`
- `ssl` boolean

**Not allowed:**
- `proxy-port` (use `full-domain` instead)

#### TCP/UDP Protocol

**Required fields:**
- `proxy-port` - Must be unique within `proxy-resources`
- Port range: 1-65535

**Not allowed:**
- `method` field in targets
- `auth` configuration (authentication not supported)
- `full-domain` (use `proxy-port` instead)

## Property Constraints

### Port Constraints

| Property | Scope | Range | Uniqueness |
|----------|-------|-------|------------|
| `proxy-port` | proxy-resources | 1-65535 | Must be unique within proxy-resources |
| `proxy-port` | client-resources | 1-65535 | Must be unique within client-resources |
| `port` (target) | All | 1-65535 | No uniqueness constraint |
| `internal-port` | All | 1-65535 | No uniqueness constraint |

**Important:** Cross-validation between proxy and client resources is NOT enforced. You can use the same port in both `proxy-resources` and `client-resources`.

### Domain Constraints

| Property | Scope | Uniqueness | Format |
|----------|-------|------------|--------|
| `full-domain` | proxy-resources (HTTP only) | Must be unique across all proxy resources | Valid domain name |

### String Length Constraints

| Property | Min Length | Max Length | Context |
|----------|------------|------------|---------|
| `name` | 2 | 100 | All resources |
| `hostname` | 1 | 255 | All targets |
| `site` | 2 | 100 | Optional site identifier |
| `header.name` | 1 | - | Header names |
| `header.value` | 1 | - | Header values |

### Authentication Constraints

#### SSO Roles

- Cannot include `"Admin"` role (reserved)
- Must be array of strings
- Each role must be valid

#### SSO Users & Whitelist Users

- Must be valid email addresses
- Must be array of strings

#### Pincode

- Must be exactly 6 digits
- Type: number
- Example: `123456`

#### Basic Auth

- Must have both `user` and `password` fields
- Both fields are required strings

## Common Validation Errors

### 1. "Admin role cannot be included in sso-roles"

**Cause:** The `Admin` role is reserved and cannot be included in `sso-roles` array.

**Solution:**
```yaml
# ❌ Wrong
auth:
  sso-enabled: true
  sso-roles:
    - Admin    # This will fail
    - Member

# ✅ Correct
auth:
  sso-enabled: true
  sso-roles:
    - Member
    - Developer
```

### 2. "Duplicate 'full-domain' values found"

**Cause:** Each `full-domain` must be unique across all proxy resources.

**Solution:**
```yaml
# ❌ Wrong - same domain twice
proxy-resources:
  app1:
    full-domain: app.example.com
    # ...
  app2:
    full-domain: app.example.com  # Duplicate!
    # ...

# ✅ Correct - use different subdomains or paths
proxy-resources:
  app1:
    full-domain: app1.example.com
    # ...
  app2:
    full-domain: app2.example.com
    # ...
```

### 3. "Duplicate 'proxy-port' values found"

**Cause:** Port numbers in `proxy-port` must be unique within their resource type.

**Solution:**
```yaml
# ❌ Wrong - same port twice in proxy-resources
proxy-resources:
  db1:
    protocol: tcp
    proxy-port: 5432
    # ...
  db2:
    protocol: tcp
    proxy-port: 5432  # Duplicate!
    # ...

# ✅ Correct - use different ports
proxy-resources:
  db1:
    protocol: tcp
    proxy-port: 5432
    # ...
  db2:
    protocol: tcp
    proxy-port: 5433
    # ...
```

### 4. "When protocol is 'http', all targets must have a 'method' field"

**Cause:** HTTP targets must specify connection method.

**Solution:**
```yaml
# ❌ Wrong - missing method
proxy-resources:
  web-app:
    protocol: http
    full-domain: app.example.com
    targets:
      - hostname: localhost
        port: 8080
        # Missing method!

# ✅ Correct - method specified
proxy-resources:
  web-app:
    protocol: http
    full-domain: app.example.com
    targets:
      - hostname: localhost
        port: 8080
        method: https  # Added method
```

### 5. "When protocol is 'tcp' or 'udp', targets must not have a 'method' field"

**Cause:** TCP/UDP targets should not include the `method` field.

**Solution:**
```yaml
# ❌ Wrong - method on TCP target
proxy-resources:
  database:
    protocol: tcp
    proxy-port: 5432
    targets:
      - hostname: localhost
        port: 5432
        method: tcp  # Not allowed for TCP/UDP!

# ✅ Correct - no method field
proxy-resources:
  database:
    protocol: tcp
    proxy-port: 5432
    targets:
      - hostname: localhost
        port: 5432
        # No method field
```

### 6. "When protocol is 'tcp' or 'udp', 'auth' must not be provided"

**Cause:** Authentication is only supported for HTTP resources.

**Solution:**
```yaml
# ❌ Wrong - auth on TCP resource
proxy-resources:
  database:
    protocol: tcp
    proxy-port: 5432
    auth:
      password: secret  # Not allowed!
    targets:
      - hostname: localhost
        port: 5432

# ✅ Correct - no auth on TCP
proxy-resources:
  database:
    protocol: tcp
    proxy-port: 5432
    # No auth field
    targets:
      - hostname: localhost
        port: 5432
```

### 7. "Resource must either be targets-only or have both 'name' and 'protocol' fields"

**Cause:** Incomplete resource definition.

**Solution:**
```yaml
# ❌ Wrong - has name but no protocol
proxy-resources:
  incomplete:
    name: My Resource
    # Missing protocol!
    targets:
      - hostname: localhost
        port: 8080

# ✅ Correct - complete definition
proxy-resources:
  complete:
    name: My Resource
    protocol: http
    full-domain: app.example.com
    targets:
      - hostname: localhost
        port: 8080
        method: http

# ✅ Also correct - targets-only
proxy-resources:
  targets-only:
    targets:
      - hostname: localhost
        port: 8080
        method: http
```

## Validation Quick Reference

### ✅ Valid Configurations

#### HTTP Resource - Complete

```yaml
proxy-resources:
  web-app:
    name: Web Application
    protocol: http
    full-domain: app.example.com
    enabled: true
    ssl: true
    host-header: backend.internal
    tls-server-name: backend.internal
    headers:
      - name: X-Custom-Header
        value: custom-value
    auth:
      sso-enabled: true
      sso-roles:
        - Member
      sso-users:
        - user@example.com
    rules:
      - action: allow
        match: ip
        value: 1.1.1.1
    targets:
      - site: site-01
        hostname: localhost
        port: 8080
        method: https
        enabled: true
        path: /api
        path-match: prefix
```

#### TCP Resource - Complete

```yaml
proxy-resources:
  database:
    name: PostgreSQL Database
    protocol: tcp
    proxy-port: 5432
    enabled: true
    targets:
      - site: site-01
        hostname: localhost
        port: 5432
        enabled: true
```

#### Targets-Only Resource

```yaml
proxy-resources:
  additional-targets:
    targets:
      - site: site-02
        hostname: backend-02
        port: 8080
        method: http
```

#### Client Resource

```yaml
client-resources:
  ssh-server:
    name: SSH Server
    protocol: tcp
    proxy-port: 2222
    hostname: localhost
    internal-port: 22
    site: site-01
    enabled: true
```

### ❌ Invalid Configurations

#### Mixed Protocol Requirements

```yaml
# ❌ INVALID - HTTP resource with proxy-port instead of full-domain
proxy-resources:
  wrong:
    name: Wrong Config
    protocol: http
    proxy-port: 8080  # Should be full-domain for HTTP!
    targets:
      - hostname: localhost
        port: 8080
        method: http
```

#### TCP with Auth

```yaml
# ❌ INVALID - TCP resource with auth
proxy-resources:
  wrong:
    name: Wrong Config
    protocol: tcp
    proxy-port: 5432
    auth:  # Auth not allowed on TCP/UDP!
      password: secret
    targets:
      - hostname: localhost
        port: 5432
```

#### Missing Required Fields

```yaml
# ❌ INVALID - HTTP target without method
proxy-resources:
  wrong:
    name: Wrong Config
    protocol: http
    full-domain: app.example.com
    targets:
      - hostname: localhost
        port: 8080
        # Missing method field!
```

## Validation Checklist

Before deploying a blueprint, verify:

- [ ] All HTTP resources have unique `full-domain` values
- [ ] All TCP/UDP resources have unique `proxy-port` values within their type
- [ ] All HTTP targets have `method` field specified
- [ ] No TCP/UDP targets have `method` field
- [ ] No TCP/UDP resources have `auth` configuration
- [ ] All port numbers are in range 1-65535
- [ ] All email addresses in `sso-users` and `whitelist-users` are valid
- [ ] SSO roles do not include "Admin"
- [ ] Pincode (if used) is exactly 6 digits
- [ ] Basic auth (if used) has both `user` and `password` fields
- [ ] String fields meet minimum/maximum length requirements
- [ ] Resources are either targets-only OR have both `name` and `protocol`
