# Role Structure

## Complete Role Directory Layout

```
role_name/
├── tasks/              # Task definitions
│   ├── main.yml       # Main entry point
│   ├── install.yml    # Installation tasks
│   ├── configure.yml  # Configuration tasks
│   └── service.yml    # Service management
├── handlers/          # Event handlers
│   └── main.yml
├── templates/         # Jinja2 templates
│   ├── config.j2
│   └── service.j2
├── files/            # Static files
│   ├── script.sh
│   └── config.conf
├── vars/             # Role variables (high precedence)
│   └── main.yml
├── defaults/         # Default variables (low precedence)
│   └── main.yml
├── meta/             # Role metadata
│   └── main.yml
├── library/          # Custom modules (optional)
│   └── custom_module.py
├── module_utils/     # Module utilities (optional)
│   └── helper.py
├── lookup_plugins/   # Custom lookups (optional)
│   └── custom_lookup.py
├── filter_plugins/   # Custom filters (optional)
│   └── custom_filter.py
├── tests/            # Test playbooks (optional)
│   ├── inventory
│   └── test.yml
└── README.md         # Documentation
```

## Directory Purposes

**tasks/** - Contains task files that define what the role does
**handlers/** - Contains handlers triggered by notify directives
**templates/** - Jinja2 templates deployed to target hosts
**files/** - Static files copied to target hosts
**vars/** - Variables with high precedence (rarely overridden)
**defaults/** - Default variables (easily overridden)
**meta/** - Role metadata including dependencies
**library/** - Custom Ansible modules
**tests/** - Test playbooks for the role

## Task Organization

**Simple role (single file):**
```yaml
# tasks/main.yml
---
- name: Install package
  ansible.builtin.package:
    name: nginx

- name: Configure service
  ansible.builtin.template:
    src: nginx.conf.j2
    dest: /etc/nginx/nginx.conf

- name: Start service
  ansible.builtin.service:
    name: nginx
    state: started
```

**Complex role (multiple files):**
```yaml
# tasks/main.yml
---
- name: Include OS-specific variables
  ansible.builtin.include_vars: "{{ ansible_os_family }}.yml"

- name: Include installation tasks
  ansible.builtin.include_tasks: install.yml

- name: Include configuration tasks
  ansible.builtin.include_tasks: configure.yml

- name: Include service tasks
  ansible.builtin.include_tasks: service.yml
```

## Variable Files

**defaults/main.yml** - Low precedence, easily overridden:
```yaml
---
nginx_port: 80
nginx_user: www-data
nginx_worker_processes: auto
```

**vars/main.yml** - High precedence, rarely overridden:
```yaml
---
nginx_config_path: /etc/nginx/nginx.conf
nginx_service_name: nginx
```

## Meta Information

**meta/main.yml:**
```yaml
---
galaxy_info:
  role_name: nginx
  author: Your Name
  description: Nginx web server role
  license: MIT
  min_ansible_version: '2.9'
  
  platforms:
    - name: Ubuntu
      versions: [focal, jammy]
    - name: EL
      versions: ['8', '9']
  
  galaxy_tags:
    - web
    - nginx

dependencies:
  - role: common
  - role: firewall
    vars:
      firewall_allowed_ports:
        - 80
        - 443

allow_duplicates: false
```

## Role Dependencies

Dependencies are executed before the role:
```yaml
dependencies:
  - role: prerequisite_role
    vars:
      var1: value1
  
  - role: another_role
    when: condition_is_true
```

## Best Practices

1. Keep tasks/ files focused and modular
2. Use defaults/ for user-configurable variables
3. Use vars/ for internal role variables
4. Document all variables in README.md
5. Include example playbook in README.md
6. Use tags for selective execution
7. Make roles idempotent
8. Test roles in isolation
