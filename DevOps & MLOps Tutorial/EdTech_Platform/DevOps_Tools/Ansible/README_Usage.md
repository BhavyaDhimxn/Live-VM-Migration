# Ansible — Usage

## 🚀 Getting Started with Ansible

This guide covers practical usage examples for Ansible in real-world automation projects.

## 📊 Example 1: Basic Playbook

### Scenario: Install and Configure Web Server

Create a playbook to install and configure nginx on multiple servers.

### Step 1: Create Inventory
```ini
# inventory.ini
[webservers]
web1.example.com
web2.example.com
web3.example.com

[webservers:vars]
ansible_user=admin
ansible_ssh_private_key_file=~/.ssh/id_rsa
```

### Step 2: Create Playbook
```yaml
# playbook.yml
- name: Install and configure nginx
  hosts: webservers
  become: yes
  tasks:
    - name: Update apt cache
      apt:
        update_cache: yes
        cache_valid_time: 3600
    
    - name: Install nginx
      apt:
        name: nginx
        state: present
    
    - name: Start and enable nginx
      systemd:
        name: nginx
        state: started
        enabled: yes
    
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

### Step 3: Run Playbook
```bash
# Test connectivity
ansible webservers -m ping

# Run playbook
ansible-playbook playbook.yml

# Run with verbose output
ansible-playbook playbook.yml -v

# Run with specific limit
ansible-playbook playbook.yml --limit web1.example.com
```

## 🔧 Example 2: Multi-server Configuration

### Scenario: Configure Database Servers

Configure PostgreSQL on database servers.

### Step 1: Create Database Playbook
```yaml
# database.yml
- name: Configure PostgreSQL
  hosts: databases
  become: yes
  vars:
    postgres_version: "13"
    db_name: "myapp"
    db_user: "appuser"
    db_password: "{{ vault_db_password }}"
  
  tasks:
    - name: Install PostgreSQL
      apt:
        name: "postgresql-{{ postgres_version }}"
        state: present
    
    - name: Create database
      postgresql_db:
        name: "{{ db_name }}"
        state: present
    
    - name: Create database user
      postgresql_user:
        name: "{{ db_user }}"
        password: "{{ db_password }}"
        priv: "{{ db_name }}:ALL"
        state: present
    
    - name: Configure PostgreSQL
      template:
        src: postgresql.conf.j2
        dest: /etc/postgresql/{{ postgres_version }}/main/postgresql.conf
      notify: restart postgresql
  
  handlers:
    - name: restart postgresql
      systemd:
        name: postgresql
        state: restarted
```

### Step 2: Run with Vault
```bash
# Run playbook with vault
ansible-playbook database.yml --ask-vault-pass
```

## 🚀 Example 3: Application Deployment

### Scenario: Deploy Application to Servers

Deploy a Python web application using Ansible.

### Step 1: Create Deployment Playbook
```yaml
# deploy.yml
- name: Deploy web application
  hosts: webservers
  become: yes
  vars:
    app_user: "appuser"
    app_dir: "/opt/myapp"
    app_version: "1.0.0"
  
  tasks:
    - name: Create application user
      user:
        name: "{{ app_user }}"
        system: yes
        shell: /bin/bash
    
    - name: Create application directory
      file:
        path: "{{ app_dir }}"
        state: directory
        owner: "{{ app_user }}"
        group: "{{ app_user }}"
        mode: '0755'
    
    - name: Install Python dependencies
      pip:
        requirements: "{{ app_dir }}/requirements.txt"
        virtualenv: "{{ app_dir }}/venv"
    
    - name: Copy application files
      copy:
        src: "{{ item }}"
        dest: "{{ app_dir }}/"
        owner: "{{ app_user }}"
        group: "{{ app_user }}"
      loop:
        - app.py
        - requirements.txt
        - config.yaml
    
    - name: Create systemd service
      template:
        src: myapp.service.j2
        dest: /etc/systemd/system/myapp.service
      notify: restart myapp
    
    - name: Enable and start service
      systemd:
        name: myapp
        enabled: yes
        state: started
        daemon_reload: yes
  
  handlers:
    - name: restart myapp
      systemd:
        name: myapp
        state: restarted
```

### Step 2: Run Deployment
```bash
# Deploy application
ansible-playbook deploy.yml

# Deploy to specific server
ansible-playbook deploy.yml --limit web1.example.com
```

## 🎯 Example 4: Using Roles

### Scenario: Reusable Configuration with Roles

Create reusable roles for common configurations.

### Step 1: Create Role Structure
```bash
# Create role structure
ansible-galaxy init roles/nginx
ansible-galaxy init roles/postgresql
ansible-galaxy init roles/python-app
```

### Step 2: Define Role Tasks
```yaml
# roles/nginx/tasks/main.yml
- name: Install nginx
  apt:
    name: nginx
    state: present

- name: Configure nginx
  template:
    src: nginx.conf.j2
    dest: /etc/nginx/nginx.conf
  notify: restart nginx

- name: Start nginx
  systemd:
    name: nginx
    state: started
    enabled: yes
```

### Step 3: Use Role in Playbook
```yaml
# site.yml
- name: Configure web servers
  hosts: webservers
  become: yes
  roles:
    - nginx
    - python-app
```

## 🔄 Example 5: Conditional Execution

### Scenario: Conditional Task Execution

Execute tasks based on conditions.

### Step 1: Create Conditional Playbook
```yaml
# conditional.yml
- name: Configure servers conditionally
  hosts: all
  become: yes
  tasks:
    - name: Install nginx (Ubuntu)
      apt:
        name: nginx
        state: present
      when: ansible_os_family == "Debian"
    
    - name: Install nginx (CentOS)
      yum:
        name: nginx
        state: present
      when: ansible_os_family == "RedHat"
    
    - name: Install development tools
      package:
        name: "{{ item }}"
        state: present
      loop:
        - git
        - curl
        - vim
      when: environment == "development"
    
    - name: Install production monitoring
      package:
        name: "{{ item }}"
        state: present
      loop:
        - prometheus
        - grafana
      when: environment == "production"
```

### Step 2: Run with Variables
```bash
# Run with environment variable
ansible-playbook conditional.yml -e "environment=production"
```

## 📊 Monitoring and Debugging

### Check Playbook Syntax
```bash
# Check playbook syntax
ansible-playbook playbook.yml --syntax-check

# Dry run (check mode)
ansible-playbook playbook.yml --check

# Diff mode (show changes)
ansible-playbook playbook.yml --check --diff
```

### Debug Playbook Execution
```bash
# Run with verbose output
ansible-playbook playbook.yml -v
ansible-playbook playbook.yml -vv
ansible-playbook playbook.yml -vvv

# Run specific task
ansible-playbook playbook.yml --tags "install"

# Skip specific task
ansible-playbook playbook.yml --skip-tags "configure"
```

## 🎯 Common Usage Patterns

### 1. **Configuration Management**
```bash
# 1. Create inventory
# inventory.ini

# 2. Create playbook
# playbook.yml

# 3. Run playbook
ansible-playbook playbook.yml
```

### 2. **Application Deployment**
```bash
# 1. Deploy application
ansible-playbook deploy.yml

# 2. Verify deployment
ansible webservers -m shell -a "systemctl status myapp"

# 3. Rollback if needed
ansible-playbook rollback.yml
```

### 3. **Multi-environment Management**
```bash
# Deploy to development
ansible-playbook deploy.yml -i inventory/dev.ini

# Deploy to production
ansible-playbook deploy.yml -i inventory/prod.ini
```

---

*Ansible is now integrated into your automation workflow! 🎉*