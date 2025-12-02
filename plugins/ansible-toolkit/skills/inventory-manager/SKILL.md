---
name: inventory-manager
description: Organizes Ansible inventory files, manages host groups, and configures dynamic inventory. Use when organizing Ansible inventory, managing host groups, or setting up dynamic inventory sources.
---

# Inventory Manager

## Quick Start

Organize Ansible inventory with proper host groups, variables, and dynamic inventory sources.

## Instructions

### Step 1: Create static inventory

**INI format:**
```ini
# inventory/production
[webservers]
web1 ansible_host=192.168.1.10
web2 ansible_host=192.168.1.11

[databases]
db1 ansible_host=192.168.1.20
db2 ansible_host=192.168.1.21

[loadbalancers]
lb1 ansible_host=192.168.1.30

[production:children]
webservers
databases
loadbalancers

[production:vars]
ansible_user=deploy
ansible_python_interpreter=/usr/bin/python3
environment=production
```

**YAML format:**
```yaml
# inventory/production.yml
all:
  children:
    production:
      children:
        webservers:
          hosts:
            web1:
              ansible_host: 192.168.1.10
            web2:
              ansible_host: 192.168.1.11
        databases:
          hosts:
            db1:
              ansible_host: 192.168.1.20
            db2:
              ansible_host: 192.168.1.21
        loadbalancers:
          hosts:
            lb1:
              ansible_host: 192.168.1.30
      vars:
        ansible_user: deploy
        environment: production
```

### Step 2: Organize group variables

**Directory structure:**
```
inventory/
├── production
├── staging
├── group_vars/
│   ├── all.yml
│   ├── webservers.yml
│   ├── databases.yml
│   └── production.yml
└── host_vars/
    ├── web1.yml
    └── db1.yml
```

**group_vars/all.yml:**
```yaml
---
# Variables for all hosts
ansible_python_interpreter: /usr/bin/python3
ntp_servers:
  - 0.pool.ntp.org
  - 1.pool.ntp.org
```

**group_vars/webservers.yml:**
```yaml
---
# Variables for webservers group
nginx_port: 80
nginx_worker_processes: 4
app_directory: /var/www/app
```

**group_vars/production.yml:**
```yaml
---
# Variables for production environment
environment: production
backup_enabled: true
monitoring_enabled: true
```

### Step 3: Configure host-specific variables

**host_vars/web1.yml:**
```yaml
---
nginx_worker_connections: 1024
server_id: 1
```

### Step 4: Use inventory in playbooks

```bash
# Run playbook with specific inventory
ansible-playbook -i inventory/production site.yml

# Target specific group
ansible-playbook -i inventory/production site.yml --limit webservers

# Target specific host
ansible-playbook -i inventory/production site.yml --limit web1
```

## Dynamic Inventory

### AWS EC2 Plugin

**inventory/aws_ec2.yml:**
```yaml
---
plugin: amazon.aws.aws_ec2
regions:
  - us-east-1
  - us-west-2

filters:
  tag:Environment: production
  instance-state-name: running

keyed_groups:
  - key: tags.Role
    prefix: role
  - key: tags.Environment
    prefix: env
  - key: placement.availability_zone
    prefix: az

hostnames:
  - tag:Name
  - private-ip-address

compose:
  ansible_host: private_ip_address
```

### Azure Plugin

**inventory/azure_rm.yml:**
```yaml
---
plugin: azure.azcollection.azure_rm
include_vm_resource_groups:
  - production-rg

keyed_groups:
  - key: tags.role
    prefix: role
  - key: tags.environment
    prefix: env

conditional_groups:
  webservers: "'web' in tags.role"
  databases: "'db' in tags.role"
```

### Custom Script

**inventory/custom.py:**
```python
#!/usr/bin/env python3
import json

inventory = {
    "webservers": {
        "hosts": ["web1", "web2"],
        "vars": {
            "nginx_port": 80
        }
    },
    "databases": {
        "hosts": ["db1", "db2"]
    },
    "_meta": {
        "hostvars": {
            "web1": {"ansible_host": "192.168.1.10"},
            "web2": {"ansible_host": "192.168.1.11"},
            "db1": {"ansible_host": "192.168.1.20"},
            "db2": {"ansible_host": "192.168.1.21"}
        }
    }
}

print(json.dumps(inventory))
```

## Inventory Patterns

**All hosts:**
```bash
ansible all -i inventory/production -m ping
```

**Specific group:**
```bash
ansible webservers -i inventory/production -m ping
```

**Multiple groups:**
```bash
ansible 'webservers:databases' -i inventory/production -m ping
```

**Exclude group:**
```bash
ansible 'all:!databases' -i inventory/production -m ping
```

**Intersection:**
```bash
ansible 'webservers:&production' -i inventory/production -m ping
```

**Regex:**
```bash
ansible '~web.*' -i inventory/production -m ping
```

## Multi-Environment Setup

```
inventory/
├── production/
│   ├── hosts
│   ├── group_vars/
│   │   ├── all.yml
│   │   └── webservers.yml
│   └── host_vars/
├── staging/
│   ├── hosts
│   ├── group_vars/
│   │   ├── all.yml
│   │   └── webservers.yml
│   └── host_vars/
└── development/
    ├── hosts
    └── group_vars/
        └── all.yml
```

**Usage:**
```bash
# Production
ansible-playbook -i inventory/production site.yml

# Staging
ansible-playbook -i inventory/staging site.yml

# Development
ansible-playbook -i inventory/development site.yml
```

## Best Practices

1. Organize inventory by environment
2. Use group_vars for shared configuration
3. Use host_vars for host-specific settings
4. Document inventory structure
5. Use dynamic inventory for cloud resources
6. Keep sensitive data in Ansible Vault
7. Use meaningful group names
8. Implement proper variable precedence
