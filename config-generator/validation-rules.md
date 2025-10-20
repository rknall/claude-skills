# Configuration Validation Rules

Comprehensive validation rules for config-generator skill.

## Critical Validation Rules

### 1. No Secrets in Config Files (CRITICAL)

**Rule**: Configuration files MUST NEVER contain secrets

**Patterns to detect**:
- PASSWORD, SECRET, KEY, TOKEN, API, AUTH, CREDENTIAL
- Long random strings (40+ chars)
- Base64-encoded values
- Private keys, certificates with keys

**Action**: If detected → Use secrets-manager skill immediately

---

### 2. .env and .env.example Synchronization (CRITICAL)

**Rule**: Files MUST have identical variable names

**Validation**:
```bash
diff <(grep -E "^[A-Z_]+" .env | cut -d'=' -f1 | sort) \
     <(grep -E "^[A-Z_]+" .env.example | cut -d'=' -f1 | sort)
```

**Action**: If mismatch → Fix synchronization before completing

---

### 3. Path Existence

**Rule**: All referenced paths must exist

**Check**:
- Volume mounts in docker-compose.yml
- File references in configs
- Directory references

---

### 4. Docker Validation

**Rule**: Always validate Docker configs

**Action**: Use docker-validation skill on docker-compose.yml

---

### 5. CLAUDE.md Requirements

**Must contain**:
- Statement: "NEVER mention Claude in commit messages"
- Stack architecture overview
- Configuration patterns
- Secrets management rules

---

## Validation Checklist

- [ ] No secrets in config files
- [ ] .env and .env.example synced
- [ ] All paths exist
- [ ] Docker validation passed
- [ ] CLAUDE.md exists with commit rule
- [ ] .gitignore has secrets/* excluded
- [ ] .dockerignore exists

---

*Strict validation ensures stack security and consistency.*
