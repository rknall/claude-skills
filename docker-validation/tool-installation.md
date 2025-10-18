# Docker Validation Tools Installation Guide

Complete installation instructions for all tools needed for Docker configuration validation.

## Required Tools

### 1. Docker & Docker Compose

#### macOS
```bash
# Install Docker Desktop (includes Docker and Compose)
brew install --cask docker

# Start Docker Desktop application
open -a Docker

# Verify installation
docker --version
docker compose version
```

#### Linux (Ubuntu/Debian)
```bash
# Install Docker Engine
curl -fsSL https://get.docker.com -o get-docker.sh
sudo sh get-docker.sh

# Add user to docker group
sudo usermod -aG docker $USER
newgrp docker

# Install Docker Compose
sudo apt-get update
sudo apt-get install docker-compose-plugin

# Verify installation
docker --version
docker compose version
```

#### Windows
```powershell
# Install Docker Desktop via Chocolatey
choco install docker-desktop

# Or download from https://www.docker.com/products/docker-desktop

# Verify installation
docker --version
docker compose version
```

**Minimum Versions:**
- Docker: 20.10.0+
- Docker Compose: v2.27.0+ (for modern syntax support)

## Validation Tools

### 2. Hadolint (Dockerfile Linter)

**Why Hadolint?**
- Parses Dockerfile into AST for deep analysis
- Integrates ShellCheck for bash validation
- Enforces Docker best practices
- Highly configurable

#### macOS
```bash
# Install via Homebrew (recommended)
brew install hadolint

# Verify installation
hadolint --version
```

#### Linux
```bash
# Download binary (x86_64)
HADOLINT_VERSION=v2.12.0
wget -O /usr/local/bin/hadolint \
  https://github.com/hadolint/hadolint/releases/download/${HADOLINT_VERSION}/hadolint-Linux-x86_64

# Make executable
chmod +x /usr/local/bin/hadolint

# Verify installation
hadolint --version
```

#### Windows
```powershell
# Download binary
$HADOLINT_VERSION = "v2.12.0"
Invoke-WebRequest -Uri "https://github.com/hadolint/hadolint/releases/download/${HADOLINT_VERSION}/hadolint-Windows-x86_64.exe" -OutFile "hadolint.exe"

# Move to PATH location
Move-Item hadolint.exe C:\Windows\System32\hadolint.exe

# Verify installation
hadolint --version
```

#### Using Docker (Cross-platform)
```bash
# Run Hadolint in container
docker run --rm -i hadolint/hadolint:latest < Dockerfile

# Create alias for convenience
echo 'alias hadolint="docker run --rm -i hadolint/hadolint:latest"' >> ~/.bashrc
source ~/.bashrc

# Verify
hadolint --version
```

#### VS Code Integration
```bash
# Install VS Code extension
code --install-extension exiasr.hadolint

# Extension will automatically use hadolint if installed
```

### 3. DCLint (Docker Compose Linter)

**Why DCLint?**
- Validates Docker Compose syntax
- Enforces best practices
- Checks for obsolete version field
- Auto-fix capabilities

#### Prerequisites
```bash
# Requires Node.js 20.19.0+
node --version

# If not installed:
# macOS
brew install node

# Linux (Ubuntu/Debian)
curl -fsSL https://deb.nodesource.com/setup_20.x | sudo -E bash -
sudo apt-get install -y nodejs

# Windows
choco install nodejs
```

#### Installation
```bash
# Install globally via npm (recommended)
npm install -g docker-compose-linter

# Or install locally in project
npm install --save-dev docker-compose-linter

# Verify installation
dclint --version
```

#### Using npx (No Installation)
```bash
# Run without global install
npx dclint docker-compose.yml
```

### 4. Trivy (Security Scanner)

**Why Trivy?**
- Comprehensive vulnerability scanner
- Scans images, filesystems, and configs
- Free and open-source
- Fast and easy to use

#### macOS
```bash
# Install via Homebrew
brew install aquasecurity/trivy/trivy

# Verify installation
trivy --version
```

#### Linux (Debian/Ubuntu)
```bash
# Add repository
sudo apt-get install wget gnupg
wget -qO - https://aquasecurity.github.io/trivy-repo/deb/public.key | gpg --dearmor | sudo tee /usr/share/keyrings/trivy.gpg > /dev/null
echo "deb [signed-by=/usr/share/keyrings/trivy.gpg] https://aquasecurity.github.io/trivy-repo/deb $(lsb_release -sc) main" | sudo tee -a /etc/apt/sources.list.d/trivy.list

# Install
sudo apt-get update
sudo apt-get install trivy

# Verify installation
trivy --version
```

#### Using Docker
```bash
# Run Trivy in container
docker run --rm -v /var/run/docker.sock:/var/run/docker.sock aquasec/trivy:latest image nginx:latest

# Scan Dockerfile
docker run --rm -v $(pwd):/app aquasec/trivy:latest config /app/Dockerfile
```

## Optional But Recommended Tools

### 5. Dive (Image Layer Analyzer)

Analyzes Docker image layers to optimize size.

```bash
# macOS
brew install dive

# Linux
wget https://github.com/wagoodman/dive/releases/download/v0.11.0/dive_0.11.0_linux_amd64.deb
sudo apt install ./dive_0.11.0_linux_amd64.deb

# Usage
dive nginx:latest
```

### 6. Docker Bench Security

Automated security audit for Docker deployments.

```bash
# Run security audit
docker run --rm --net host --pid host --userns host --cap-add audit_control \
    -e DOCKER_CONTENT_TRUST=$DOCKER_CONTENT_TRUST \
    -v /etc:/etc:ro \
    -v /usr/bin/containerd:/usr/bin/containerd:ro \
    -v /usr/bin/runc:/usr/bin/runc:ro \
    -v /usr/lib/systemd:/usr/lib/systemd:ro \
    -v /var/lib:/var/lib:ro \
    -v /var/run/docker.sock:/var/run/docker.sock:ro \
    --label docker_bench_security \
    docker/docker-bench-security
```

### 7. Snyk (Alternative Security Scanner)

```bash
# Install Snyk CLI
npm install -g snyk

# Authenticate
snyk auth

# Test Dockerfile
snyk container test nginx:latest

# Test Docker image
snyk container test myapp:latest --file=Dockerfile
```

## Configuration Files Setup

### Create .hadolint.yaml
```bash
cat > .hadolint.yaml << 'EOF'
---
ignored:
  - DL3008  # Pin versions in apt-get (sometimes too strict)
  - DL3013  # Pin versions in pip
  - DL3018  # Pin versions in apk

override:
  error:
    - DL3002  # Last user should not be root
    - DL3020  # Use COPY instead of ADD
  warning:
    - DL3003  # Use WORKDIR instead of cd

trustedRegistries:
  - docker.io
  - ghcr.io
  - gcr.io
  - quay.io

strict-labels: true
EOF
```

### Create .dclintrc.json
```bash
cat > .dclintrc.json << 'EOF'
{
  "rules": {
    "no-version-field": "error",
    "require-quotes": "warning",
    "service-name-case": ["error", "kebab-case"],
    "no-duplicate-keys": "error",
    "require-restart": "warning"
  },
  "exclude": [
    "node_modules/**",
    ".git/**",
    "dist/**"
  ]
}
EOF
```

### Create .dockerignore
```bash
cat > .dockerignore << 'EOF'
# Git
.git
.gitignore
.gitattributes

# CI/CD
.github
.gitlab-ci.yml
Jenkinsfile

# Documentation
README.md
CHANGELOG.md
docs/

# Dependencies
node_modules/
vendor/
__pycache__/
*.pyc

# Build artifacts
dist/
build/
target/
*.log

# IDE
.vscode/
.idea/
*.swp
*.swo

# Environment
.env
.env.local
*.pem
*.key
EOF
```

## Verification Script

Create a script to verify all tools are installed:

```bash
#!/bin/bash
# verify-tools.sh

echo "🔍 Checking Docker Validation Tools..."
echo ""

# Check Docker
if command -v docker &> /dev/null; then
    echo "✅ Docker: $(docker --version)"
else
    echo "❌ Docker: NOT INSTALLED"
fi

# Check Docker Compose
if docker compose version &> /dev/null; then
    echo "✅ Docker Compose: $(docker compose version)"
else
    echo "❌ Docker Compose: NOT INSTALLED"
fi

# Check Hadolint
if command -v hadolint &> /dev/null; then
    echo "✅ Hadolint: $(hadolint --version)"
else
    echo "⚠️  Hadolint: NOT INSTALLED (recommended)"
fi

# Check DCLint
if command -v dclint &> /dev/null; then
    echo "✅ DCLint: $(dclint --version 2>&1 | head -1)"
else
    echo "⚠️  DCLint: NOT INSTALLED (recommended)"
fi

# Check Trivy
if command -v trivy &> /dev/null; then
    echo "✅ Trivy: $(trivy --version | head -1)"
else
    echo "⚠️  Trivy: NOT INSTALLED (optional)"
fi

# Check Node.js (for DCLint)
if command -v node &> /dev/null; then
    echo "✅ Node.js: $(node --version)"
else
    echo "⚠️  Node.js: NOT INSTALLED (needed for DCLint)"
fi

echo ""
echo "📊 Installation Summary:"
echo "   Required tools: Docker, Docker Compose"
echo "   Recommended: Hadolint, DCLint"
echo "   Optional: Trivy, Dive, Docker Bench"
```

Make it executable and run:
```bash
chmod +x verify-tools.sh
./verify-tools.sh
```

## Quick Start Commands

After installation, test the tools:

```bash
# Test Hadolint
echo 'FROM node:latest' > test.Dockerfile
hadolint test.Dockerfile
rm test.Dockerfile

# Test DCLint
echo 'services:
  web:
    image: nginx:latest' > test-compose.yml
dclint test-compose.yml
rm test-compose.yml

# Test Docker Compose validation
docker compose config --quiet

# Test Trivy (if installed)
trivy image nginx:latest

# Test full validation
./validate-docker.sh  # (if you have the validation script)
```

## Troubleshooting

### Hadolint: Command not found
```bash
# Check if binary exists
ls -la /usr/local/bin/hadolint

# Check PATH
echo $PATH

# Add to PATH if needed
export PATH="/usr/local/bin:$PATH"
echo 'export PATH="/usr/local/bin:$PATH"' >> ~/.bashrc
```

### DCLint: Module not found
```bash
# Check npm global prefix
npm config get prefix

# If not in PATH, add it
export PATH="$(npm config get prefix)/bin:$PATH"

# Or use npx
npx dclint docker-compose.yml
```

### Docker Compose: version command not recognized
```bash
# Check Docker Compose version
docker compose version  # New (v2)
docker-compose --version  # Old (v1, deprecated)

# If using v1, upgrade to v2
# Follow Docker documentation for your OS
```

### Permission Denied (Linux)
```bash
# Add user to docker group
sudo usermod -aG docker $USER
newgrp docker

# Verify
docker ps
```

## CI/CD Integration

### GitHub Actions
Already includes all tools in standard ubuntu-latest runner:
```yaml
- name: Validate Docker configs
  run: |
    docker --version
    docker compose version
    # Install additional tools if needed
```

### GitLab CI
```yaml
before_script:
  - apt-get update
  - apt-get install -y wget
  - wget -O /usr/local/bin/hadolint https://github.com/hadolint/hadolint/releases/download/v2.12.0/hadolint-Linux-x86_64
  - chmod +x /usr/local/bin/hadolint
```

## Update Tools

Keep tools up to date:

```bash
# Update Hadolint (macOS)
brew upgrade hadolint

# Update DCLint
npm update -g docker-compose-linter

# Update Trivy (macOS)
brew upgrade trivy

# Update Docker Desktop
# Use built-in updater or:
brew upgrade --cask docker
```

---

**All tools installed? Run `./verify-tools.sh` to confirm!**
