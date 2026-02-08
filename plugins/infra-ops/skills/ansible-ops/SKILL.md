---
name: ansible-ops
description: Ansible operations patterns covering playbook execution, secret management with 1Password, task automation, and security best practices. Use when writing or running Ansible playbooks.
---

# Ansible Operations

## Running Playbooks

Use a task runner (like mise, make, or just) to wrap playbook execution and handle environment setup:

```bash
# Preferred: Use task runner that loads env automatically
mise run playbook ansible/playbooks/deploy-service-stack.yml

# With extra ansible-playbook arguments
mise run playbook ansible/playbooks/deploy-monitoring-stack.yml --ask-become-pass

# Direct execution (ensure env vars are loaded first)
ansible-playbook -i ansible/inventory/hosts.ini ansible/playbooks/deploy-service.yml
```

## Secret Management with 1Password

### Service Account Pattern

Store the 1Password service account token in a `.env` file (gitignored) and load it automatically through your task runner:

```bash
# .env (NEVER commit this)
OP_SERVICE_ACCOUNT_TOKEN=ops_xxxxx...
```

```bash
# Verify authentication
op whoami
# Should show: User Type: SERVICE_ACCOUNT
```

### Using Secrets in Playbooks

```yaml
- name: Deploy service with secrets
  environment:
    API_KEY: "{{ lookup('env', 'OP_SERVICE_ACCOUNT_TOKEN') | community.general.onepassword_raw('API Key', vault='MyVault') }}"
```

**Important notes:**
- The `community.general.onepassword` lookup may not support service accounts directly
- Alternative: Use `op` CLI directly in playbooks with the environment token
- Always use your task runner instead of running ansible-playbook directly to ensure tokens are loaded

### Security Rules

- The `.env` file must be in `.gitignore` - **NEVER commit it**
- Service account tokens can be rotated in your 1Password admin console
- Use `no_log: true` in Ansible tasks that handle secrets:

```yaml
- name: Set secret environment variable
  ansible.builtin.lineinfile:
    path: /etc/environment
    line: "SECRET_KEY={{ secret_value }}"
  no_log: true
```

## Playbook Best Practices

### Task Organization

```yaml
- name: Deploy web service
  hosts: web_servers
  become: true
  tasks:
    # 1. Create required directories
    - name: Ensure config directory exists
      ansible.builtin.file:
        path: /opt/myservice/config
        state: directory
        owner: appuser
        group: appgroup
        mode: "0755"

    # 2. Deploy configuration files
    - name: Copy service configuration
      ansible.builtin.template:
        src: templates/service.conf.j2
        dest: /opt/myservice/config/service.conf
        owner: appuser
        group: appgroup
        mode: "0644"
      notify: restart service

    # 3. Deploy the service
    - name: Deploy docker stack
      community.docker.docker_stack:
        name: myservice
        compose:
          - /opt/myservice/docker-compose.yml
```

### Idempotency

Every task should be safe to run multiple times:

```yaml
# GOOD: Idempotent - creates only if missing
- name: Create data directory
  ansible.builtin.file:
    path: /data/myservice
    state: directory
    mode: "0755"

# BAD: Not idempotent - appends every run
- name: Add config line
  ansible.builtin.shell: echo "setting=value" >> /etc/config
```

### Explicit Permissions

Always set permissions explicitly:

```yaml
- name: Deploy config file
  ansible.builtin.copy:
    src: files/config.yaml
    dest: /opt/service/config.yaml
    owner: appuser
    group: appgroup
    mode: "0644"  # Always set mode explicitly
```

### Handlers

Use handlers for service restarts:

```yaml
handlers:
  - name: restart service
    community.docker.docker_stack:
      name: myservice
      compose:
        - /opt/myservice/docker-compose.yml
      state: present

  - name: reload nginx
    ansible.builtin.service:
      name: nginx
      state: reloaded
```

## Inventory Management

```ini
# ansible/inventory/hosts.ini
[web_servers]
web-01 ansible_host=10.0.0.10
web-02 ansible_host=10.0.0.11

[db_servers]
db-01 ansible_host=10.0.0.20

[gpu_nodes]
gpu-01 ansible_host=10.0.0.30

[all:vars]
ansible_user=deploy
ansible_become=true
```

## Common Patterns

### Copy Files from Repo

```yaml
- name: Deploy compose file
  ansible.builtin.copy:
    src: "{{ playbook_dir }}/../docker/stacks/myservice/docker-compose.yml"
    dest: /opt/myservice/docker-compose.yml
    mode: "0644"
```

### Template with Variables

```yaml
- name: Deploy templated config
  ansible.builtin.template:
    src: templates/config.yml.j2
    dest: /opt/service/config.yml
    mode: "0644"
  vars:
    db_host: "{{ hostvars['db-01'].ansible_host }}"
    app_port: 8080
```

### Wait for Service

```yaml
- name: Wait for service to be healthy
  ansible.builtin.uri:
    url: "http://localhost:8080/health"
    status_code: 200
  register: result
  until: result.status == 200
  retries: 30
  delay: 5
```

## Debugging

```bash
# Dry run (check mode)
ansible-playbook playbook.yml --check

# Verbose output
ansible-playbook playbook.yml -vvv

# Limit to specific hosts
ansible-playbook playbook.yml --limit web-01

# List tasks without executing
ansible-playbook playbook.yml --list-tasks

# Start at a specific task
ansible-playbook playbook.yml --start-at-task="Deploy config"
```
