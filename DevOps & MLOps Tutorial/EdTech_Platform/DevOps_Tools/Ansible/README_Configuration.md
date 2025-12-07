# Ansible — Configuration

## ⚙️ Configuration Overview

Ansible configuration is managed through the `ansible.cfg` file and environment variables to customize behavior for your specific needs.

## 📁 Configuration Files

### 1. **Ansible Configuration** (`ansible.cfg`)
```ini
[defaults]
# Inventory file
inventory = inventory.ini

# Remote user
remote_user = ansible

# Private key file
private_key_file = ~/.ssh/id_rsa

# Host key checking
host_key_checking = False

# Retry files
retry_files_enabled = False

# Roles path
roles_path = ~/.ansible/roles:/usr/share/ansible/roles:/etc/ansible/roles

# Logging
log_path = /var/log/ansible.log

[privilege_escalation]
become = True
become_method = sudo
become_user = root
become_ask_pass = False

[ssh_connection]
ssh_args = -C -o ControlMaster=auto -o ControlPersist=60s
pipelining = True
```

### 2. **Environment Variables**
```bash
# Set Ansible configuration file
export ANSIBLE_CONFIG=/path/to/ansible.cfg

# Set inventory file
export ANSIBLE_INVENTORY=/path/to/inventory

# Set host key checking
export ANSIBLE_HOST_KEY_CHECKING=False
```

## 🔧 Basic Configuration

### 1. **Inventory Configuration**
```ini
# inventory.ini
[webservers]
web1.example.com ansible_host=192.168.1.10
web2.example.com ansible_host=192.168.1.11

[webservers:vars]
ansible_user=admin
ansible_ssh_private_key_file=~/.ssh/id_rsa

[databases]
db1.example.com ansible_host=192.168.1.20

[databases:vars]
ansible_user=admin
ansible_ssh_private_key_file=~/.ssh/id_rsa

[all:vars]
ansible_python_interpreter=/usr/bin/python3
```

### 2. **SSH Configuration**
```ini
# ansible.cfg
[ssh_connection]
ssh_args = -C -o ControlMaster=auto -o ControlPersist=60s
pipelining = True
control_path = ~/.ansible/cp/%%h-%%p-%%r
```

### 3. **Privilege Escalation**
```ini
# ansible.cfg
[privilege_escalation]
become = True
become_method = sudo
become_user = root
become_ask_pass = False
```

## 🔐 Security Configuration

### 1. **SSH Key Management**
```bash
# Generate SSH key
ssh-keygen -t ed25519 -C "ansible@example.com"

# Copy SSH key to hosts
ssh-copy-id user@hostname

# Or use Ansible
ansible all -m authorized_key -a "user=ansible key='{{ lookup('file', '~/.ssh/id_ed25519.pub') }}'"
```

### 2. **Vault for Secrets**
```bash
# Create encrypted vault file
ansible-vault create secrets.yml

# Edit vault file
ansible-vault edit secrets.yml

# Use in playbook
ansible-playbook playbook.yml --ask-vault-pass
```

### 3. **Credential Management**
```yaml
# Use Ansible Vault
# secrets.yml (encrypted)
db_password: !vault |
  $ANSIBLE_VAULT;1.1;AES256
  663864396539663133393364363836313...

# Use in playbook
- name: Configure database
  vars_files:
    - secrets.yml
  tasks:
    - name: Set database password
      set_fact:
        db_pass: "{{ db_password }}"
```

## 🔄 Advanced Configuration

### 1. **Dynamic Inventory**
```python
# aws_ec2.py (dynamic inventory)
#!/usr/bin/env python
import boto3
import json

ec2 = boto3.client('ec2')
instances = ec2.describe_instances()

inventory = {
    'webservers': {
        'hosts': [],
        'vars': {}
    }
}

for reservation in instances['Reservations']:
    for instance in reservation['Instances']:
        if instance['State']['Name'] == 'running':
            inventory['webservers']['hosts'].append(instance['PublicIpAddress'])

print(json.dumps(inventory))
```

### 2. **Callback Plugins**
```ini
# ansible.cfg
[defaults]
callback_plugins = /path/to/callback_plugins
stdout_callback = yaml
```

### 3. **Connection Plugins**
```ini
# ansible.cfg
[defaults]
connection_plugins = /path/to/connection_plugins
transport = ssh
```

## 🐛 Common Configuration Issues

### Issue 1: Host Key Checking
```bash
# Error: Host key verification failed
# Solution: Disable host key checking
export ANSIBLE_HOST_KEY_CHECKING=False
# Or in ansible.cfg:
[defaults]
host_key_checking = False
```

### Issue 2: Python Interpreter
```bash
# Error: Python interpreter not found
# Solution: Set Python interpreter
ansible all -m setup -a "filter=ansible_python_version"
# Or in inventory:
[all:vars]
ansible_python_interpreter=/usr/bin/python3
```

### Issue 3: Permission Denied
```bash
# Error: Permission denied
# Solution: Configure privilege escalation
[privilege_escalation]
become = True
become_method = sudo
```

## ✅ Configuration Validation

### Test Configuration
```bash
# Test inventory
ansible-inventory --list

# Test connectivity
ansible all -m ping

# Test with specific host
ansible web1.example.com -m ping
```

### Configuration Checklist
- [ ] ansible.cfg configured
- [ ] Inventory file created
- [ ] SSH keys configured
- [ ] Host key checking configured
- [ ] Privilege escalation configured
- [ ] Vault configured (if using secrets)

## 🔧 Configuration Best Practices

1. **🔐 Security**: Use Ansible Vault for secrets
2. **📊 Organization**: Organize inventory by environment
3. **🔄 Automation**: Use dynamic inventory for cloud
4. **📝 Documentation**: Document all configuration changes
5. **🧪 Testing**: Test configuration in development first

---

*Your Ansible configuration is now optimized for automation! 🎯*