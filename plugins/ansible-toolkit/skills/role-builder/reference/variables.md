# Variables

## Variable Precedence (lowest to highest)

1. role defaults (defaults/main.yml)
2. inventory file or script group vars
3. inventory group_vars/all
4. playbook group_vars/all
5. inventory group_vars/*
6. playbook group_vars/*
7. inventory file or script host vars
8. inventory host_vars/*
9. playbook host_vars/*
10. host facts / cached set_facts
11. play vars
12. play vars_prompt
13. play vars_files
14. role vars (vars/main.yml)
15. block vars
16. task vars
17. include_vars
18. set_facts / registered vars
19. role (and include_role) params
20. include params
21. extra vars (-e in CLI)

## Role Variables

**defaults/main.yml** - User-configurable defaults:
```yaml
---
# Application settings
app_port: 8080
app_user: appuser
app_directory: /opt/myapp

# Feature flags
enable_monitoring: false
enable_backups: true
```

**vars/main.yml** - Internal role variables:
```yaml
---
# Paths (don't override)
app_config_path: "{{ app_directory }}/config"
app_log_path: "/var/log/{{ app_user }}"

# Computed values
app_full_path: "{{ app_directory }}/{{ app_version }}"
```

## Variable Definition

**In playbook:**
```yaml
---
- hosts: webservers
  vars:
    nginx_port: 8080
    nginx_user: www-data
  roles:
    - nginx
```

**In inventory:**
```ini
[webservers]
web1 ansible_host=192.168.1.10 nginx_port=8080
web2 ansible_host=192.168.1.11 nginx_port=8081

[webservers:vars]
nginx_user=www-data
```

**In group_vars:**
```yaml
# group_vars/webservers.yml
---
nginx_port: 8080
nginx_worker_processes: 4
```

**In host_vars:**
```yaml
# host_vars/web1.yml
---
nginx_port: 8080
nginx_worker_connections: 1024
```

## Variable Types

**Simple variables:**
```yaml
app_name: myapp
app_port: 8080
```

**Lists:**
```yaml
packages:
  - nginx
  - python3
  - git
```

**Dictionaries:**
```yaml
database:
  host: localhost
  port: 5432
  name: mydb
  user: dbuser
```

**Nested structures:**
```yaml
application:
  name: myapp
  version: 1.0.0
  config:
    port: 8080
    workers: 4
    database:
      host: localhost
      port: 5432
```

## Variable Usage

**In tasks:**
```yaml
- name: Install {{ app_name }}
  ansible.builtin.package:
    name: "{{ app_name }}"

- name: Create directory
  ansible.builtin.file:
    path: "{{ app_directory }}"
    owner: "{{ app_user }}"
```

**In templates:**
```jinja2
# config.j2
server {
    listen {{ nginx_port }};
    server_name {{ server_name }};
    
    location / {
        proxy_pass http://{{ backend_host }}:{{ backend_port }};
    }
}
```

## Variable Filters

**String manipulation:**
```yaml
- name: Uppercase
  ansible.builtin.debug:
    msg: "{{ app_name | upper }}"

- name: Default value
  ansible.builtin.debug:
    msg: "{{ undefined_var | default('default_value') }}"
```

**List operations:**
```yaml
- name: Join list
  ansible.builtin.debug:
    msg: "{{ packages | join(', ') }}"

- name: First item
  ansible.builtin.debug:
    msg: "{{ packages | first }}"
```

**Dictionary operations:**
```yaml
- name: Get keys
  ansible.builtin.debug:
    msg: "{{ database | dict2items }}"
```

## Registered Variables

```yaml
- name: Check if file exists
  ansible.builtin.stat:
    path: /etc/myapp/config
  register: config_file

- name: Create config if missing
  ansible.builtin.template:
    src: config.j2
    dest: /etc/myapp/config
  when: not config_file.stat.exists
```

## Facts

**Gathering facts:**
```yaml
- name: Gather facts
  ansible.builtin.setup:

- name: Use facts
  ansible.builtin.debug:
    msg: "OS: {{ ansible_distribution }} {{ ansible_distribution_version }}"
```

**Custom facts:**
```yaml
- name: Set custom fact
  ansible.builtin.set_fact:
    app_environment: production
    cacheable: true

- name: Use custom fact
  ansible.builtin.debug:
    msg: "Environment: {{ app_environment }}"
```

## Best Practices

1. Use defaults/ for user-configurable variables
2. Use vars/ for internal role variables
3. Document all variables in README.md
4. Use meaningful variable names
5. Group related variables
6. Use default filter for optional variables
7. Validate variable types and values
8. Avoid overriding facts
