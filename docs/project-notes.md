# Ansible Infrastructure Lab — Project Notes

## Architecture

This project uses one Ansible Control Node to configure and manage two Ubuntu web servers.

```text
                    Ansible Control Node
                            |
                            | SSH
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

Ansible connects to both Managed Nodes using the dedicated `ansible` user and SSH public-key authentication.

Privileged operations are performed using `sudo` through:

```yaml
become: true
```

Management flow:

```text
Control Node
     |
     | SSH Public Key
     v
ansible user
     |
     | sudo / become
     v
Privileged Configuration
```

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
│   ├── users/
│   ├── nginx/
│   └── security/
│
└── docs/
    └── project-notes.md
```

---

## Roles

The infrastructure is divided into reusable Ansible Roles:

```text
site.yml
   |
   +-- common
   |
   +-- users
   |
   +-- nginx
   |
   +-- security
```

### common

Responsible for baseline server configuration:

* APT cache management
* Common package installation
* Timezone configuration
* Application directory creation
* Ansible Facts usage
* OS-based conditions

### users

Responsible for Linux user configuration:

* DevOps user creation
* `sudo` group membership
* Home directory management
* `.ssh` directory management
* SSH public-key deployment

### nginx

Responsible for web server configuration:

* Nginx installation
* Service management
* Custom web root
* Jinja2 templates
* Host-specific web pages
* Nginx site configuration
* Handlers
* `nginx -t` validation
* Graceful reload
* HTTP verification

### security

Responsible for basic firewall configuration:

* UFW installation
* Allow SSH: `22/tcp`
* Allow HTTP: `80/tcp`
* Allow HTTPS: `443/tcp`
* Default deny incoming traffic
* Default allow outgoing traffic

---

## Main Playbook

The complete infrastructure is managed through:

```text
playbooks/site.yml
```

The playbook runs the roles in this order:

```text
common
   ↓
users
   ↓
nginx
   ↓
security
```

Full deployment:

```bash
ansible-playbook playbooks/site.yml
```

Run only one part of the infrastructure:

```bash
ansible-playbook playbooks/site.yml --tags nginx
```

```bash
ansible-playbook playbooks/site.yml --tags users
```

```bash
ansible-playbook playbooks/site.yml --tags security
```

---

## Validation Workflow

Before applying infrastructure changes:

```bash
ansible-playbook playbooks/site.yml --syntax-check
```

Review tasks:

```bash
ansible-playbook playbooks/site.yml --list-tasks
```

Dry run:

```bash
ansible-playbook playbooks/site.yml --check
```

Dry run with configuration differences:

```bash
ansible-playbook playbooks/site.yml --check --diff
```

Apply:

```bash
ansible-playbook playbooks/site.yml
```

Run again to verify idempotency:

```bash
ansible-playbook playbooks/site.yml
```

The desired result after convergence is approximately:

```text
changed=0
failed=0
unreachable=0
```

---

# Troubleshooting

## SSH / Unreachable

Typical error:

```text
UNREACHABLE!
Permission denied (publickey,password)
```

Test SSH independently:

```bash
ssh ansible@SERVER_IP
```

Check Ansible with verbose output:

```bash
ansible all -m ping -vvv
```

Verify:

* SSH username
* Private key
* `authorized_keys`
* SSH permissions
* Inventory IP address

---

## Inventory Problems

Inspect the inventory:

```bash
ansible-inventory --graph
```

Detailed view:

```bash
ansible-inventory --list
```

Common problems:

* Wrong IP
* Wrong group
* Wrong `ansible_user`
* Wrong SSH key path

---

## YAML / Playbook Errors

Check syntax:

```bash
ansible-playbook playbooks/site.yml --syntax-check
```

Common causes:

* Wrong indentation
* Missing `:`
* Invalid list structure
* Incorrect module parameters

---

## Undefined Variables

Typical error:

```text
The task includes an option with an undefined variable
```

Inspect a variable:

```bash
ansible webservers -m debug -a "var=VARIABLE_NAME"
```

Check:

* `group_vars`
* Role `defaults`
* Variable spelling
* Variable precedence

---

## sudo / become Problems

Test privilege escalation:

```bash
ansible webservers -b -m command -a "whoami"
```

Expected:

```text
root
```

Check:

* User belongs to `sudo`
* sudoers configuration
* `become: true`

---

## Nginx Problems

Validate configuration:

```bash
ansible webservers -b -m command -a "nginx -t"
```

Check service:

```bash
ansible webservers -b -m command -a "systemctl is-active nginx"
```

Check deployed configuration:

```bash
ansible webservers -b -m command -a "cat /etc/nginx/sites-available/devops-lab"
```

Test HTTP:

```bash
curl -v http://192.168.1.20
```

```bash
curl -v http://192.168.1.30
```

---

## Firewall Problems

Check UFW:

```bash
ansible webservers -b -m command -a "ufw status verbose"
```

Expected allowed ports:

```text
22/tcp
80/tcp
443/tcp
```

Always allow SSH before enabling a default-deny incoming firewall policy.

---

## Verbose Debugging

Increase verbosity when the normal error is not enough:

```bash
ansible-playbook playbooks/site.yml -v
```

```bash
ansible-playbook playbooks/site.yml -vv
```

```bash
ansible-playbook playbooks/site.yml -vvv
```

`-vvv` is especially useful for SSH, connection and module execution problems.

---

## Troubleshooting Flow

```text
Failure
   |
   v
Read the failed task
   |
   v
Identify the layer
   |
   +-- Inventory
   +-- SSH
   +-- sudo
   +-- Variables
   +-- Module
   +-- Service
   +-- HTTP / Application
   |
   v
Test that layer independently
   |
   v
Use verbose output if required
   |
   v
Fix
   |
   v
Run the playbook again
   |
   v
Verify idempotency
```

---

## Key Concepts Practiced

This project demonstrates:

* Inventory and host groups
* SSH key authentication
* Privilege escalation
* Ad-hoc commands
* Playbooks and tasks
* Ansible modules
* Variables and `group_vars`
* Variable precedence
* Ansible Facts
* Conditions
* Loops
* Desired state
* Idempotency
* User management
* Package management
* File management
* Jinja2 templates
* Handlers and notifications
* Nginx service management
* HTTP verification
* Ansible Roles
* Tags
* Check Mode
* Diff Mode
* UFW firewall management
* Troubleshooting
