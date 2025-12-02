# Handlers

## Handler Basics

Handlers are tasks triggered by `notify` directives, typically used for service restarts or reloads.

**Defining handlers:**
```yaml
# handlers/main.yml
---
- name: restart nginx
  ansible.builtin.service:
    name: nginx
    state: restarted

- name: reload nginx
  ansible.builtin.service:
    name: nginx
    state: reloaded
```

**Triggering handlers:**
```yaml
# tasks/main.yml
---
- name: Update nginx configuration
  ansible.builtin.template:
    src: nginx.conf.j2
    dest: /etc/nginx/nginx.conf
  notify: restart nginx
```

## Handler Execution

- Handlers run at the end of the play
- Handlers run only once, even if notified multiple times
- Handlers run in the order they're defined, not notified
- Use `meta: flush_handlers` to run handlers immediately

**Immediate execution:**
```yaml
- name: Update config
  ansible.builtin.template:
    src: config.j2
    dest: /etc/app/config
  notify: restart app

- name: Run handlers now
  ansible.builtin.meta: flush_handlers

- name: Verify app is running
  ansible.builtin.uri:
    url: http://localhost:8080/health
```

## Handler Patterns

**Multiple handlers:**
```yaml
- name: Update configuration
  ansible.builtin.template:
    src: nginx.conf.j2
    dest: /etc/nginx/nginx.conf
  notify:
    - validate nginx config
    - reload nginx
```

**Conditional handlers:**
```yaml
- name: restart nginx
  ansible.builtin.service:
    name: nginx
    state: restarted
  when: nginx_restart_required | default(false)
```

**Handler with validation:**
```yaml
- name: validate nginx config
  ansible.builtin.command:
    cmd: nginx -t
  changed_when: false

- name: reload nginx
  ansible.builtin.service:
    name: nginx
    state: reloaded
```

## Listen Directive

Group handlers under a common topic:

```yaml
# handlers/main.yml
---
- name: restart web services
  ansible.builtin.service:
    name: nginx
    state: restarted
  listen: "restart web stack"

- name: restart app services
  ansible.builtin.service:
    name: myapp
    state: restarted
  listen: "restart web stack"

# tasks/main.yml
---
- name: Update configuration
  ansible.builtin.template:
    src: config.j2
    dest: /etc/config
  notify: "restart web stack"
```

## Best Practices

1. Use descriptive handler names
2. Validate before restarting services
3. Use reload instead of restart when possible
4. Group related handlers with listen
5. Use flush_handlers for critical operations
6. Make handlers idempotent
7. Handle failures gracefully
