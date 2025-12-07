# Git — Usage

## 🚀 Getting Started with Git

This guide covers practical usage examples for Git in real-world development workflows.

## 📊 Example 1: Basic Repository Workflow

### Scenario: Create and Manage a Repository

Let's create a new repository and perform basic Git operations.

### Step 1: Initialize Repository
```bash
# Create project directory
mkdir my-project && cd my-project

# Initialize Git repository
git init

# Check status
git status
```

### Step 2: Make First Commit
```bash
# Create initial file
echo "# My Project" > README.md

# Stage file
git add README.md

# Commit changes
git commit -m "Initial commit: Add README"

# View commit history
git log
```

### Step 3: Make Additional Changes
```bash
# Create new file
echo "console.log('Hello World');" > app.js

# Stage all changes
git add .

# Commit with descriptive message
git commit -m "Add JavaScript application file"

# View changes
git log --oneline
```

## 🔧 Example 2: Branching Workflow

### Scenario: Develop Features in Branches

Use branches to develop features independently.

### Step 1: Create Feature Branch
```bash
# Create and switch to feature branch
git checkout -b feature/user-authentication

# Make changes
echo "function login() {}" > auth.js
git add auth.js
git commit -m "Add user authentication feature"
```

### Step 2: Switch Between Branches
```bash
# Switch back to main
git checkout main

# Create another feature branch
git checkout -b feature/payment-processing

# Make changes
echo "function processPayment() {}" > payment.js
git add payment.js
git commit -m "Add payment processing feature"
```

### Step 3: Merge Feature Branch
```bash
# Switch to main
git checkout main

# Merge feature branch
git merge feature/user-authentication

# View merged branches
git branch --merged

# Delete merged branch
git branch -d feature/user-authentication
```

## 🚀 Example 3: Working with Remote Repositories

### Scenario: Collaborate with Remote Repository

Work with GitHub, GitLab, or other remote repositories.

### Step 1: Add Remote Repository
```bash
# Add remote repository
git remote add origin https://github.com/user/repo.git

# View remotes
git remote -v

# Fetch from remote
git fetch origin
```

### Step 2: Push to Remote
```bash
# Push main branch to remote
git push -u origin main

# Push feature branch
git push -u origin feature-branch

# Push all branches
git push --all origin
```

### Step 3: Pull from Remote
```bash
# Pull latest changes
git pull origin main

# Fetch and merge separately
git fetch origin
git merge origin/main
```

## 🎯 Example 4: Resolving Merge Conflicts

### Scenario: Handle Merge Conflicts

Resolve conflicts when merging branches.

### Step 1: Create Conflict Scenario
```bash
# On main branch
echo "Version 1" > file.txt
git add file.txt
git commit -m "Add file.txt"

# On feature branch
git checkout -b feature-branch
echo "Version 2" > file.txt
git add file.txt
git commit -m "Update file.txt on feature branch"

# Back to main
git checkout main
echo "Version 3" > file.txt
git add file.txt
git commit -m "Update file.txt on main"
```

### Step 2: Attempt Merge
```bash
# Merge feature branch (will create conflict)
git merge feature-branch

# Git will show conflict
# <<<<<<< HEAD
# Version 3
# =======
# Version 2
# >>>>>>> feature-branch
```

### Step 3: Resolve Conflict
```bash
# Edit file to resolve conflict
# Choose one version or combine both

# After resolving, stage the file
git add file.txt

# Complete the merge
git commit -m "Merge feature-branch: Resolve conflict"
```

## 🔄 Example 5: Git Workflow Patterns

### Scenario: Git Flow Workflow

Implement Git Flow for release management.

### Step 1: Initialize Git Flow
```bash
# Install git-flow (optional)
# macOS: brew install git-flow
# Linux: sudo apt-get install git-flow

# Initialize git-flow
git flow init

# Or manually create branches
git checkout -b develop
git checkout -b main
```

### Step 2: Feature Development
```bash
# Start feature
git checkout develop
git checkout -b feature/new-feature

# Develop feature
# ... make changes ...
git add .
git commit -m "Implement new feature"

# Finish feature
git checkout develop
git merge feature/new-feature
git branch -d feature/new-feature
```

### Step 3: Release Management
```bash
# Start release
git checkout develop
git checkout -b release/1.0.0

# Prepare release
# ... finalize changes ...
git add .
git commit -m "Prepare release 1.0.0"

# Finish release
git checkout main
git merge release/1.0.0
git tag -a v1.0.0 -m "Release version 1.0.0"
git checkout develop
git merge release/1.0.0
git branch -d release/1.0.0
```

## 📊 Monitoring and Debugging

### Check Repository Status
```bash
# Check working directory status
git status

# View changes
git diff

# View staged changes
git diff --staged

# View commit history
git log --oneline --graph --all
```

### Debug Issues
```bash
# View file history
git log --follow file.txt

# Find when bug was introduced
git bisect start
git bisect bad
git bisect good <commit-hash>
# Git will help find the problematic commit

# View what changed
git show <commit-hash>
```

## 🎯 Common Usage Patterns

### 1. **Daily Development Workflow**
```bash
# 1. Pull latest changes
git pull origin main

# 2. Create feature branch
git checkout -b feature/my-feature

# 3. Make changes
# ... edit files ...

# 4. Stage and commit
git add .
git commit -m "Implement feature"

# 5. Push to remote
git push origin feature/my-feature

# 6. Create pull request
# (On GitHub/GitLab web interface)
```

### 2. **Code Review Workflow**
```bash
# 1. Create review branch
git checkout -b review/feature-name

# 2. Make changes based on feedback
# ... edit files ...

# 3. Commit fixes
git add .
git commit -m "Address review comments"

# 4. Push updates
git push origin review/feature-name
```

### 3. **Hotfix Workflow**
```bash
# 1. Create hotfix from main
git checkout main
git checkout -b hotfix/critical-bug

# 2. Fix bug
# ... fix code ...

# 3. Commit and merge
git add .
git commit -m "Fix critical bug"
git checkout main
git merge hotfix/critical-bug
git tag -a v1.0.1 -m "Hotfix version 1.0.1"
```

---

*Git is now integrated into your development workflow! 🎉*