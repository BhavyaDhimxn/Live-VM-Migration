# Git — Best Practices

## 🎯 Production Best Practices

### 1. **Repository Structure**
```
project/
├── .git/
├── .gitignore
├── README.md
├── src/
│   ├── app.py
│   └── utils.py
├── tests/
│   └── test_app.py
├── docs/
│   └── api.md
└── config/
    └── settings.yaml
```

### 2. **Commit Message Convention**
```bash
# Use conventional commit messages
git commit -m "feat: Add user authentication"
git commit -m "fix: Resolve login bug"
git commit -m "docs: Update API documentation"
git commit -m "refactor: Improve code structure"
git commit -m "test: Add unit tests for auth"

# Format: <type>(<scope>): <subject>
# Types: feat, fix, docs, style, refactor, test, chore
```

## 🔐 Security Best Practices

### 1. **Never Commit Secrets**
```bash
# Create .gitignore
cat > .gitignore << EOF
# Secrets
.env
*.key
*.pem
secrets/
credentials.json

# Environment files
.env.local
.env.production

# API keys
config/api_keys.yaml
EOF

# If secrets were committed, remove them
git rm --cached secrets.txt
git commit -m "Remove secrets from repository"
```

### 2. **Use .gitignore Effectively**
```bash
# Comprehensive .gitignore
cat > .gitignore << EOF
# Dependencies
node_modules/
venv/
__pycache__/
*.pyc

# Build artifacts
dist/
build/
*.egg-info/

# IDE files
.vscode/
.idea/
*.swp
*.swo

# OS files
.DS_Store
Thumbs.db

# Logs
*.log
logs/
EOF
```

### 3. **Branch Protection**
```bash
# Protect main branch (on GitHub/GitLab)
# Settings > Branches > Add rule
# - Require pull request reviews
# - Require status checks
# - Require branches to be up to date
# - Restrict who can push
```

## 📊 Commit Best Practices

### 1. **Atomic Commits**
```bash
# Good: Small, focused commits
git add src/auth.py
git commit -m "Add user authentication module"

git add tests/test_auth.py
git commit -m "Add tests for authentication"

# Bad: Large, unrelated changes
git add .
git commit -m "Update everything"
```

### 2. **Descriptive Commit Messages**
```bash
# Good: Clear and descriptive
git commit -m "fix: Resolve memory leak in data processing

- Fix memory leak in process_data function
- Add proper resource cleanup
- Update error handling"

# Bad: Vague messages
git commit -m "fix stuff"
git commit -m "update"
```

### 3. **Commit Frequency**
```bash
# Commit often, push regularly
# - Commit after completing a logical unit of work
# - Push to remote at least daily
# - Don't let local commits accumulate for weeks
```

## 🔄 Branching Best Practices

### 1. **Branch Naming Convention**
```bash
# Use consistent naming
feature/user-authentication
bugfix/login-error
hotfix/security-patch
release/1.0.0
chore/update-dependencies

# Avoid:
my-branch
test
fix
```

### 2. **Keep Branches Short-Lived**
```bash
# Merge branches quickly
# - Don't let feature branches live for months
# - Merge to main/develop regularly
# - Delete merged branches

git branch -d feature/merged-feature
```

### 3. **Branch Strategy**
```bash
# Choose a workflow:
# - Git Flow: For projects with releases
# - GitHub Flow: For continuous deployment
# - GitLab Flow: For environment-based deployments

# Stick to one strategy per project
```

## 🔄 CI/CD Integration

### 1. **Git Hooks**
```bash
# Pre-commit hook
cat > .git/hooks/pre-commit << 'EOF'
#!/bin/sh
# Run tests before commit
npm test
if [ $? -ne 0 ]; then
    echo "Tests failed. Commit aborted."
    exit 1
fi
EOF

chmod +x .git/hooks/pre-commit
```

### 2. **Webhook Integration**
```bash
# Configure webhooks on GitHub/GitLab
# - Push events trigger CI/CD
# - Pull request events trigger tests
# - Tag events trigger deployments
```

## 📈 Code Review Best Practices

### 1. **Pull Request Guidelines**
```bash
# Create focused pull requests
# - One feature per PR
# - Keep PRs small and reviewable
# - Include description and screenshots
# - Link related issues
```

### 2. **Review Process**
```bash
# Review checklist:
# - Code follows style guide
# - Tests are included
# - Documentation is updated
# - No secrets are committed
# - Changes are tested
```

## ⚡ Performance Optimization

### 1. **Repository Optimization**
```bash
# Clean up repository
git gc --aggressive

# Remove large files from history
git filter-branch --tree-filter 'rm -f large-file.txt' HEAD

# Use Git LFS for large files
git lfs install
git lfs track "*.psd"
git lfs track "*.zip"
```

### 2. **Shallow Clones**
```bash
# Clone with limited history
git clone --depth 1 https://github.com/user/repo.git

# Fetch full history later if needed
git fetch --unshallow
```

## 🔧 Reproducibility Best Practices

### 1. **Tag Releases**
```bash
# Tag releases properly
git tag -a v1.0.0 -m "Release version 1.0.0"
git tag -a v1.0.0 -m "Release version 1.0.0" <commit-hash>

# Push tags
git push origin v1.0.0
git push origin --tags
```

### 2. **Documentation**
```bash
# Keep README updated
# Document:
# - Setup instructions
# - Development workflow
# - Contribution guidelines
# - Release process
```

## 🧪 Testing Best Practices

### 1. **Test Before Commit**
```bash
# Run tests locally
npm test
pytest

# Only commit if tests pass
git add .
git commit -m "feat: Add new feature"
```

### 2. **CI Integration**
```yaml
# .github/workflows/test.yml
name: Tests
on: [push, pull_request]
jobs:
  test:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v2
    - run: npm test
```

## 📚 Documentation Best Practices

### 1. **README.md**
```markdown
# Project Name

## Description
Brief description of the project

## Installation
```bash
npm install
```

## Usage
```bash
npm start
```

## Contributing
Guidelines for contributing

## License
MIT
```

### 2. **CHANGELOG.md**
```markdown
# Changelog

## [1.0.0] - 2023-01-01
### Added
- User authentication
- Payment processing

### Fixed
- Login bug
- Memory leak
```

## 🚨 Common Pitfalls to Avoid

### 1. **Don't Commit Secrets**
```bash
# Bad: Committing secrets
git add .env
git commit -m "Add configuration"

# Good: Use .gitignore
echo ".env" >> .gitignore
git add .gitignore
git commit -m "Add .gitignore"
```

### 2. **Don't Force Push to Main**
```bash
# Bad: Force push to main
git push --force origin main

# Good: Use force push only on feature branches
git push --force origin feature-branch
```

### 3. **Don't Commit Generated Files**
```bash
# Bad: Committing build artifacts
git add dist/ build/ node_modules/

# Good: Ignore generated files
echo "dist/" >> .gitignore
echo "build/" >> .gitignore
echo "node_modules/" >> .gitignore
```

---

*Follow these best practices to maintain a clean and efficient Git workflow! 🎯*