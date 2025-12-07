# Ansible — Best Practices

## 🎯 Production Best Practices

### 1. **Playbook Organization**
```
ansible-project/
├── playbooks/
│   ├── site.yml
│   ├── webservers.yml
│   └── databases.yml
├── roles/
│   ├── nginx/
│   ├── postgresql/
│   └── python-app/
├── inventory/
│   ├── production.ini
│   ├── staging.ini
│   └── development.ini
├── group_vars/
│   ├── all.yml
│   ├── webservers.yml
│   └── databases.yml
├── host_vars/
│   └── web1.example.com.yml
├── ansible.cfg
└── requirements.yml
```

### 2. **Playbook Naming Convention**
```yaml
# Use descriptive names
# Format: {purpose}.yml
# Examples:
# - deploy-webapp.yml
# - configure-database.yml
# - setup-monitoring.yml
```

## 🔐 Security Best Practices

### 1. **Secrets Management**
```yaml
# Use Ansible Vault for secrets
# Never commit unencrypted secrets

# Create vault file
ansible-vault create group_vars/all/secrets.yml

# Encrypt existing file
ansible-vault encrypt secrets.yml

# Use in playbook
- name: Configure database
  vars_files:
    - group_vars/all/secrets.yml
  tasks:
    - name: Set database password
      set_fact:
        db_password: "{{ vault_db_password }}"
```

### 2. **SSH Key Management**
```bash
# Use SSH keys, not passwords
# Generate dedicated Ansible key
ssh-keygen -t ed25519 -f ~/.ssh/ansible_key -C "ansible@example.com"

# Distribute keys securely
ansible all -m authorized_key -a "user=ansible key='{{ lookup('file', '~/.ssh/ansible_key.pub') }}'"
```

### 3. **Limit Privilege Escalation**
```yaml
# Use become only when necessary
- name: Install package
  apt:
    name: nginx
    state: present
  become: yes  # Only when needed

# Use specific become_user
- name: Configure application
  template:
    src: config.j2
    dest: /opt/app/config.yaml
  become: yes
  become_user: appuser
```

## 📊 Playbook Best Practices

### 1. **Idempotent Tasks**
```yaml
# Make tasks idempotent
- name: Ensure nginx is installed
  apt:
    name: nginx
    state: present  # Idempotent

- name: Ensure nginx is running
  systemd:
    name: nginx
    state: started  # Idempotent
```

### 2. **Use Handlers**
```yaml
# Use handlers for service restarts
tasks:
  - name: Configure nginx
    template:
      src: nginx.conf.j2
      dest: /etc/nginx/nginx.conf
    notify: restart nginx

handlers:
  - name: restart nginx
    systemd:
      name: nginx
      state: restarted
```

### 3. **Error Handling**
```yaml
# Implement error handling
- name: Install package
  apt:
    name: nginx
    state: present
  ignore_errors: yes
  register: install_result

- name: Check installation
  fail:
    msg: "Package installation failed"
  when: install_result.rc != 0
```

## 🔄 CI/CD Integration

### 1. **Automated Playbook Execution**
```yaml
# .github/workflows/ansible-deploy.yml
name: Ansible Deployment

on:
  push:
    branches: [main]

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v2
    
    - name: Setup Ansible
      run: |
        pip install ansible
    
    - name: Run playbook
      env:
        ANSIBLE_VAULT_PASSWORD: ${{ secrets.VAULT_PASSWORD }}
      run: |
        echo "$ANSIBLE_VAULT_PASSWORD" > .vault_pass
        ansible-playbook deploy.yml --vault-password-file .vault_pass
```

## 📈 Monitoring Best Practices

### 1. **Playbook Logging**
```yaml
# Enable logging
# ansible.cfg
[defaults]
log_path = /var/log/ansible.log
```

### 2. **Task Timing**
```yaml
# Measure task execution time
- name: Install packages
  apt:
    name: "{{ packages }}"
  register: install_result

- name: Display timing
  debug:
    msg: "Installation took {{ install_result.elapsed }} seconds"
```

## ⚡ Performance Optimization

### 1. **Parallel Execution**
```bash
# Run tasks in parallel
ansible-playbook playbook.yml -f 10  # 10 parallel forks
```

### 2. **Pipelining**
```ini
# Enable pipelining in ansible.cfg
[ssh_connection]
pipelining = True
```

### 3. **Fact Caching**
```ini
# Enable fact caching
[defaults]
fact_caching = jsonfile
fact_caching_connection = /tmp/ansible_facts
```

## 🔧 Reproducibility Best Practices

### 1. **Version Control**
```bash
# Version control playbooks
git init
git add playbooks/ roles/ inventory/
git commit -m "Add Ansible playbooks"
```

### 2. **Role Versioning**
```yaml
# requirements.yml
roles:
  - name: nginx
    src: geerlingguy.nginx
    version: 3.1.0
```

## 🧪 Testing Best Practices

### 1. **Syntax Checking**
```bash
# Check playbook syntax
ansible-playbook playbook.yml --syntax-check

# Check in CI/CD
ansible-lint playbook.yml
```

### 2. **Idempotency Testing**
```bash
# Run playbook twice to test idempotency
ansible-playbook playbook.yml
ansible-playbook playbook.yml  # Should show no changes
```

## 📚 Documentation Best Practices

### 1. **Playbook Documentation**
```yaml
---
# Playbook: Deploy web application
# Description: Deploys Python web application to web servers
# Author: DevOps Team
# Last Updated: 2023-01-01

- name: Deploy web application
  hosts: webservers
  become: yes
  vars:
    app_version: "1.0.0"
  # ...
```

### 2. **Role Documentation**
```yaml
# roles/nginx/meta/main.yml
galaxy_info:
  author: DevOps Team
  description: Install and configure nginx
  license: MIT
  min_ansible_version: 2.9
```

## 🚨 Common Pitfalls to Avoid

### 1. **Don't Use Shell Module Unnecessarily**
```yaml
# Bad: Use shell for simple tasks
- name: Install package
  shell: apt-get install nginx

# Good: Use appropriate module
- name: Install package
  apt:
    name: nginx
    state: present
```

### 2. **Don't Ignore Idempotency**
```yaml
# Bad: Not idempotent
- name: Create file
  shell: echo "content" > /tmp/file.txt

# Good: Idempotent
- name: Create file
  copy:
    content: "content"
    dest: /tmp/file.txt
```

### 3. **Don't Hardcode Values**
```yaml
# Bad: Hardcoded values
- name: Install nginx
  apt:
    name: nginx

# Good: Use variables
- name: Install {{ web_server }}
  apt:
    name: "{{ web_server }}"
```

---

*Follow these best practices to build robust automation with Ansible! 🎯*