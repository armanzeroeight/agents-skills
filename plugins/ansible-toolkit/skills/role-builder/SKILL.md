---
name: role-builder
description: Creates Ansible roles with proper structure, tasks, handlers, and variables. Use when creating Ansible roles, organizing automation tasks, or structuring configuration management.
---

# Role Builder

## Quick Start

Create well-structured Ansible roles with proper organization, reusability, and idempotency.

## Instructions

### Step 1: Initialize role structure

```bash
# Create role using ansible-galaxy
ansible-galaxy init my_role

# Or create manually
mkdir -p roles/my_role/{tasks,handlers,templates,files,vars,defaults,meta}
```

**Standard role structure:**
```
my_role/
├── tasks/
│   └── main.yml          # Main task list
├── handlers/
│   └── main.yml          # Handlers
├── templates/
│   └── config.j2         # Jinja2 templates
├── files/
│   └── script.sh         # Static files
├── vars/
│   └── main.yml          # Role variables
├── defaults/
│   └── main.yml          # Default variables
├── meta/
│   └── main.yml          # Role metadata and dependencies
└── README.md             # Role documentation
```

### Step 2: Define tasks

**tasks/main.yml:**
```yaml
---
# Main task file for my_role

- name: Install required packages
  ansible.builtin.package:
    name: "{{ item }}"
    state: present
  loop: "{{ required_packages }}"
  tags: [packages]

- name: Create application directory
  ansible.builtin.file:
    path: "{{ app_directory }}"
    state: directory
    owner: "{{ app_user }}"
    group: "{{ app_group }}"
    mode: '0755'
  tags: [setup]

- name: Deploy configuration file
  ansible.builtin.template:
    src: config.j2
    dest: "{{ app_directory }}/config.yml"
    owner: "{{ app_user }}"
    group: "{{ app_group }}"
    mode: '0644'
  notify: restart application
  tags: [config]

- name: Ensure service is running
  ansible.builtin.service:
    name: "{{ service_name }}"
    state: started
    enabled: true
  tags: [service]
```

### Step 3: Create handlers

**handlers/main.yml:**
```yaml
---
# Handlers for my_role

- name: restart application
  ansible.builtin.service:
    name: "{{ service_name }}"
    state: restarted

- name: reload application
  ansible.builtin.service:
    name: "{{ service_name }}"
    state: reloaded

- name: validate configuration
  ansible.builtin.command:
    cmd: "{{ app_binary }} --validate-config"
  changed_when: false
```

### Step 4: Define variables

**defaults/main.yml (default values):**
```yaml
---
# Default variables for my_role

app_user: appuser
app_group: appgroup
app_directory: /opt/myapp
service_name: myapp

required_packages:
  - python3
  - python3-pip
  - git
```

**vars/main.yml (role-specific variables):**
```yaml
---
# Role-specific variables

app_binary: /usr/local/bin/myapp
config_template: config.j2
```

### Step 5: Add metadata

**meta/main.yml:**
```yaml
---
galaxy_info:
  author: Your Name
  description: Role for deploying my application
  company: Your Company
  license: MIT
  min_ansible_version: '2.9'
  
  platforms:
    - name: Ubuntu
      versions:
        - focal
        - jammy
    - name: EL
      versions:
        - '8'
        - '9'
  
  galaxy_tags:
    - application
    - deployment

dependencies:
  - role: common
    vars:
      common_packages:
        - curl
        - wget
```

## Role Design Patterns

### Pattern 1: Modular tasks

**Break tasks into separate files:**
```yaml
# tasks/main.yml
---
- name: Include installation tasks
  ansible.builtin.include_tasks: install.yml
  tags: [install]

- name: Include configuration tasks
  ansible.builtin.include_tasks: configure.yml
  tags: [configure]

- name: Include service tasks
  ansible.builtin.include_tasks: service.yml
  tags: [service]
```

### Pattern 2: Conditional execution

```yaml
- name: Install on Debian-based systems
  ansible.builtin.apt:
    name: "{{ package_name }}"
    state: present
  when: ansible_os_family == "Debian"

- name: Install on RedHat-based systems
  ansible.builtin.yum:
    name: "{{ package_name }}"
    state: present
  when: ansible_os_family == "RedHat"
```

### Pattern 3: Idempotent operations

```yaml
- name: Ensure configuration is present
  ansible.builtin.lineinfile:
    path: /etc/myapp/config
    line: "{{ config_line }}"
    state: present
    create: true
  # Idempotent - only changes if line is missing

- name: Ensure service is running
  ansible.builtin.service:
    name: myapp
    state: started
  # Idempotent - only starts if not running
```

## Best Practices

**Use fully qualified collection names:**
```yaml
# Good
- name: Install package
  ansible.builtin.package:
    name: nginx

# Avoid
- name: Install package
  package:
    name: nginx
```

**Add descriptive names:**
```yaml
# Good
- name: Install nginx web server
  ansible.builtin.package:
    name: nginx

# Avoid
- name: Install
  ansible.builtin.package:
    name: nginx
```

**Use tags for selective execution:**
```yaml
- name: Install packages
  ansible.builtin.package:
    name: "{{ item }}"
  loop: "{{ packages }}"
  tags: [packages, install]

- name: Configure application
  ansible.builtin.template:
    src: config.j2
    dest: /etc/app/config
  tags: [config, configure]
```

**Implement error handling:**
```yaml
- name: Attempt risky operation
  ansible.builtin.command:
    cmd: /usr/local/bin/risky-command
  register: result
  failed_when: false
  changed_when: result.rc == 0
  tags: [risky]

- name: Handle failure
  ansible.builtin.debug:
    msg: "Operation failed, continuing anyway"
  when: result.rc != 0
```

## Advanced

For detailed information, see:
- [Role Structure](reference/role-structure.md) - Complete role directory layout and organization
- [Handlers](reference/handlers.md) - Handler patterns and best practices
- [Variables](reference/variables.md) - Variable precedence and management
