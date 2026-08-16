# Ansible Linux Infrastructure Automation

A hands-on DevOps project for automating the configuration and management of multiple Ubuntu servers using Ansible.

This project starts with SSH connectivity and inventory management and evolves into a role-based Ansible architecture for server baseline configuration, user management, Nginx deployment, SSH access management, firewall configuration, and infrastructure validation.

## Architecture

```text
                    Ansible Control Node
                            |
                            | SSH
                    User: ansible
                            |
                 +----------+----------+
                 |                     |
                 v                     v
               web01                 web02
           192.168.1.20          192.168.1.30
                 |                     |
              Ubuntu                 Ubuntu
                 |                     |
               Nginx                 Nginx
```

Ansible connects to the Managed Nodes using SSH public-key authentication.

Privileged operations use:

```yaml
become: true
```

This allows Ansible to connect using the non-root `ansible` user and use `sudo` only when administrative privileges are required.

---

## Project Goals

This project demonstrates practical use of Ansible for:

* Managing multiple Linux servers
* SSH key-based authentication
* Inventory and host groups
* Server baseline configuration
* Package management
* Linux user management
* SSH authorized key deployment
* Ansible Facts and conditions
* Variables and variable precedence
* Nginx installation and configuration
* Jinja2 templates
* Handlers and notifications
* HTTP service verification
* UFW firewall management
* Loops
* Tags
* Check Mode and Diff Mode
* Reusable Ansible Roles
* Idempotent configuration management
* Troubleshooting common Ansible failures

---

## Project Structure

```text
ansible-infrastructure-lab/
├── ansible.cfg
├── requirements.yml
├── README.md
├── .gitignore
│
├── inventory/
│   ├── hosts.ini
│   └── group_vars/
│       └── webservers.yml
│
├── playbooks/
│   └── site.yml
│
├── roles/
│   ├── common/
│   │   ├── defaults/
│   │   │   └── main.yml
│   │   └── tasks/
│   │       └── main.yml
│   │
│   ├── users/
│   │   ├── defaults/
│   │   │   └── main.yml
│   │   └── tasks/
│   │       └── main.yml
│   │
│   ├── nginx/
│   │   ├── defaults/
│   │   │   └── main.yml
│   │   ├── tasks/
│   │   │   └── main.yml
│   │   ├── handlers/
│   │   │   └── main.yml
│   │   └── templates/
│   │       ├── index.html.j2
│   │       └── nginx-site.conf.j2
│   │
│   └── security/
│       ├── defaults/
│       │   └── main.yml
│       └── tasks/
│           └── main.yml
│
└── docs/
    └── project-notes.md
```

---

## Managed Nodes

| Host  | IP Address   | Group      | Role             |
| ----- | ------------ | ---------- | ---------------- |
| web01 | 192.168.1.20 | webservers | Nginx Web Server |
| web02 | 192.168.1.30 | webservers | Nginx Web Server |

---

## Ansible Configuration

The project uses a local `ansible.cfg`:

```ini
[defaults]
inventory = inventory/hosts.ini
roles_path = ./roles
host_key_checking = True
interpreter_python = auto_silent
```

This configuration:

* Uses the project inventory by default
* Defines the local Ansible Roles directory
* Keeps SSH host key verification enabled
* Automatically discovers the remote Python interpreter

---

## Inventory

```ini
[webservers]
web01 ansible_host=192.168.1.20
web02 ansible_host=192.168.1.30

[webservers:vars]
ansible_user=ansible
ansible_ssh_private_key_file=~/.ssh/id_ed25519
```

Verify the inventory:

```bash
ansible-inventory --graph
```

Test connectivity:

```bash
ansible all -m ping
```

---

## Roles

### common

Provides baseline Linux configuration:

* APT cache management
* Common package installation
* Timezone configuration
* Application directory creation
* Ansible Facts
* OS-family conditions

### users

Manages Linux users and SSH access:

* Creates the DevOps user
* Configures `/bin/bash`
* Adds the user to the `sudo` group
* Creates the `.ssh` directory
* Deploys an SSH public key using `authorized_key`

### nginx

Manages the web-server layer:

* Nginx installation
* Service management
* Custom web root
* Host-specific HTML deployment
* Jinja2 Nginx configuration
* Custom site activation
* Default site removal
* Configuration validation with `nginx -t`
* Handler-based Nginx reload
* HTTP endpoint verification

### security

Provides basic firewall configuration:

* UFW installation
* Allow `22/tcp`
* Allow `80/tcp`
* Allow `443/tcp`
* Default deny incoming traffic
* Default allow outgoing traffic
* Firewall activation

---

## Main Playbook

The complete infrastructure is managed through:

```text
playbooks/site.yml
```

Role execution order:

```text
common
   ↓
users
   ↓
nginx
   ↓
security
```

Run the complete infrastructure configuration:

```bash
ansible-playbook playbooks/site.yml
```

---

## Dependencies

Required Ansible Collections are defined in:

```text
requirements.yml
```

Install them with:

```bash
ansible-galaxy collection install -r requirements.yml
```

Collections used by the project:

* `community.general`
* `ansible.posix`

---

## Validation Workflow

### Syntax Check

```bash
ansible-playbook playbooks/site.yml --syntax-check
```

### List Tasks

```bash
ansible-playbook playbooks/site.yml --list-tasks
```

### List Tags

```bash
ansible-playbook playbooks/site.yml --list-tags
```

### Check Mode

```bash
ansible-playbook playbooks/site.yml --check
```

### Check Mode with Diff

```bash
ansible-playbook playbooks/site.yml --check --diff
```

### Apply Configuration

```bash
ansible-playbook playbooks/site.yml
```

A typical workflow is:

```text
Write
  ↓
Syntax Check
  ↓
Check Mode
  ↓
Review Diff
  ↓
Apply
  ↓
Verify
```

---

## Tags

Individual roles can be executed independently.

Common configuration:

```bash
ansible-playbook playbooks/site.yml --tags common
```

User management:

```bash
ansible-playbook playbooks/site.yml --tags users
```

Nginx:

```bash
ansible-playbook playbooks/site.yml --tags nginx
```

Security:

```bash
ansible-playbook playbooks/site.yml --tags security
```

Multiple roles:

```bash
ansible-playbook playbooks/site.yml --tags nginx,security
```

---

## Jinja2 Templates

Nginx configuration and HTML pages are generated dynamically using Jinja2.

Example:

```nginx
listen {{ nginx_port }};
root {{ nginx_web_root }};
```

Host-specific information can also be rendered using inventory variables and Ansible Facts:

```html
<p>Server: {{ inventory_hostname }}</p>
<p>System Hostname: {{ ansible_hostname }}</p>
<p>Environment: {{ lab_environment }}</p>
```

The same template therefore generates different content for `web01` and `web02`.

---

## Handlers

Nginx is reloaded only when configuration changes.

```text
Template changed
       ↓
notify
       ↓
nginx -t
       ↓
Configuration valid
       ↓
Reload Nginx
```

If no configuration change occurs, the handler is not triggered.

---

## Idempotency

The project describes the desired state of the infrastructure rather than repeatedly executing raw shell commands.

After the infrastructure reaches the required state, subsequent executions should result in approximately:

```text
changed=0
failed=0
unreachable=0
```

This demonstrates Ansible idempotency.

---

## Verification

Test Ansible connectivity:

```bash
ansible all -m ping
```

Verify privilege escalation:

```bash
ansible webservers -b -m command -a "whoami"
```

Expected result:

```text
root
```

Verify Nginx:

```bash
ansible webservers -b -m command -a "systemctl is-active nginx"
```

Validate Nginx configuration:

```bash
ansible webservers -b -m command -a "nginx -t"
```

Test HTTP:

```bash
curl http://192.168.1.20
curl http://192.168.1.30
```

Verify UFW:

```bash
ansible webservers -b -m command -a "ufw status verbose"
```

---

## Troubleshooting

Use verbose output when required:

```bash
ansible all -m ping -vvv
```

```bash
ansible-playbook playbooks/site.yml -vvv
```

Inspect inventory:

```bash
ansible-inventory --graph
```

```bash
ansible-inventory --list
```

Common troubleshooting areas include:

* SSH authentication
* Inventory configuration
* Role discovery and `roles_path`
* sudo / become
* Undefined variables
* YAML syntax
* Nginx configuration
* Service failures
* Firewall rules

Additional notes are available in:

```text
docs/project-notes.md
```

---

## Development Journey

The project was intentionally developed incrementally before being refactored into Roles.

```text
SSH & Inventory
       ↓
Baseline Playbook
       ↓
Variables & Facts
       ↓
User Management
       ↓
Nginx Automation
       ↓
Jinja2 & Handlers
       ↓
Reusable Roles
       ↓
Firewall & Security
       ↓
Final site.yml
```

---

## Security Notes

SSH private keys must never be committed to Git.

The repository only references the local SSH private-key path.

Sensitive local files are excluded using `.gitignore`.

---

## Project Status

| Phase   | Description                           | Status    |
| ------- | ------------------------------------- | --------- |
| Phase 1 | Connectivity and Inventory            | Completed |
| Phase 2 | Baseline Configuration                | Completed |
| Phase 3 | Variables, Facts and User Management  | Completed |
| Phase 4 | Nginx, Jinja2 Templates and Handlers  | Completed |
| Phase 5 | Roles, Security and Final Refactoring | Completed |

**Project completed successfully.**

---

## Skills Practiced

* Ansible Inventory
* Ad-hoc Commands
* Playbooks
* Modules
* Variables
* Group Variables
* Facts
* Conditions
* Loops
* Idempotency
* User Management
* SSH Key Management
* Package Management
* File Management
* Service Management
* Jinja2 Templates
* Handlers
* Roles
* Tags
* Check Mode
* Diff Mode
* Nginx
* UFW
* Linux Administration
* Troubleshooting

---

## Purpose

This repository is a practical DevOps learning and portfolio project demonstrating how Ansible can automate repeatable Linux server configuration and web-server deployment using maintainable, role-based infrastructure automation.
