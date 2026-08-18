# Ansible Playbooks

## What is an Ansible Playbook?

Ansible Playbooks are Ansible's **orchestration language**.

A playbook defines **what Ansible should do, which hosts it should perform the work on, and in what sequence the tasks should be executed**.

A playbook is written in **YAML**.

A simple way to think about it:

```text
Playbook
   |
   +-- Play 1
   |     |
   |     +-- Task 1
   |     +-- Task 2
   |
   +-- Play 2
         |
         +-- Task 1
         +-- Task 2
```

---

## Playbook Structure

An Ansible playbook is normally a single YAML file containing one or more **plays**.

```yaml
---
- name: Play 1
  hosts: localhost
  tasks:

    - name: Execute command 'date'
      ansible.builtin.command: date

    - name: Execute script on server
      ansible.builtin.script: test_script.sh

- name: Play 2
  hosts: webservers
  tasks:

    - name: Install web service
      ansible.builtin.yum:
        name: httpd
        state: present

    - name: Start web server
      ansible.builtin.service:
        name: httpd
        state: started
```

> The `ansible.builtin.*` form is the fully qualified collection name (FQCN). It makes the module being used explicit.

---

## Playbook

A **playbook** is the complete YAML file containing one or more plays.

Example:

```text
site.yml
```

The playbook describes the overall automation workflow.

---

## Play

A **play** maps a set of tasks to a particular group of hosts.

```yaml
- name: Configure web servers
  hosts: webservers
  tasks:
    ...
```

A play can be thought of as:

> "Perform these tasks against these hosts."

A playbook can contain multiple plays targeting different hosts.

---

## Task

A **task** is an individual unit of work inside a play.

```yaml
- name: Install Apache
  ansible.builtin.yum:
    name: httpd
    state: present
```

Each task normally invokes a module to perform the requested action.

---

## Module

A **module** performs the actual operation requested by a task.

Examples:

| Module | Purpose |
|---|---|
| `command` | Execute a command |
| `shell` | Execute a command through a shell |
| `script` | Transfer and execute a local script |
| `yum` | Manage packages on YUM-based systems |
| `dnf` | Manage packages using DNF |
| `service` | Manage services |
| `copy` | Copy files to managed hosts |
| `file` | Manage files, directories and permissions |
| `user` | Manage users |
| `debug` | Display information during execution |

Ansible provides hundreds of modules. The official Ansible module documentation should be used to check available parameters and behavior.

---

## Understanding the Relationship

```text
Playbook
    |
    +-- Play
          |
          +-- hosts
          |
          +-- tasks
                |
                +-- Task
                      |
                      +-- Module
```

For example:

```yaml
- name: Install Apache
  hosts: webservers

  tasks:
    - name: Install Apache package
      ansible.builtin.yum:
        name: httpd
        state: present
```

Here:

- **Playbook** → the YAML file
- **Play** → `Install Apache`
- **Hosts** → `webservers`
- **Task** → `Install Apache package`
- **Module** → `ansible.builtin.yum`
- **Module parameters** → `name` and `state`

---

## The `hosts` Parameter

The `hosts` parameter specifies which inventory hosts the play should run against.

```yaml
- name: Configure web servers
  hosts: webservers
  tasks:
    ...
```

If the inventory contains:

```ini
[webservers]
web1
web2
web3
```

then the play targets all three hosts.

```text
Play
 |
 +-- web1
 +-- web2
 +-- web3
```

The target hosts are selected at the **play level**.

---

## Multiple Plays in One Playbook

```yaml
---
- name: Configure web servers
  hosts: webservers

  tasks:
    - name: Install Apache
      ansible.builtin.yum:
        name: httpd
        state: present

- name: Configure database servers
  hosts: databases

  tasks:
    - name: Install PostgreSQL
      ansible.builtin.yum:
        name: postgresql-server
        state: present
```

This allows one playbook to orchestrate changes across different parts of an infrastructure.

---

## Common Operations Performed by Playbooks

Playbooks can automate operations such as:

- Execute commands
- Run scripts
- Install or remove packages
- Start, stop or restart services
- Create users
- Copy configuration files
- Modify files
- Create directories
- Manage permissions
- Configure applications
- Deploy software
- Restart systems when required

Example:

```yaml
- name: Configure application server
  hosts: appservers

  tasks:

    - name: Install required package
      ansible.builtin.yum:
        name: httpd
        state: present

    - name: Start web service
      ansible.builtin.service:
        name: httpd
        state: started

    - name: Create application directory
      ansible.builtin.file:
        path: /opt/myapp
        state: directory
        mode: '0755'
```

---

## Running a Playbook

Basic syntax:

```bash
ansible-playbook <playbook-file>
```

Example:

```bash
ansible-playbook playbook.yml
```

With a custom inventory:

```bash
ansible-playbook -i inventory.ini playbook.yml
```

---

## Getting Help

```bash
ansible-playbook --help
```

This shows additional command-line options without requiring them to be memorized.

---

## Checking Playbook Syntax

Before running a playbook:

```bash
ansible-playbook --syntax-check playbook.yml
```

This is useful for catching YAML/Ansible syntax problems before execution.

---

## Listing Hosts

```bash
ansible-playbook -i inventory.ini playbook.yml --list-hosts
```

This is useful when working with groups and larger inventories.

---

## Listing Tasks

```bash
ansible-playbook playbook.yml --list-tasks
```

This shows the tasks contained in the playbook without executing them.

---

## Ad-hoc Commands vs Playbooks

### Ad-hoc command

```bash
ansible localhost -m ansible.builtin.command -a "date"
```

Ad-hoc commands are useful for:

- Quick tests
- One-time operations
- Checking connectivity
- Simple troubleshooting

### Playbook

```bash
ansible-playbook -i inventory.ini playbook.yml
```

Playbooks are better for:

- Multiple tasks
- Repeatable automation
- Structured configuration
- Variables
- Conditions
- Loops
- Handlers
- Multiple plays
- Version-controlled automation

A useful mental model:

```text
Ad-hoc command
    ↓
Quick / one-off operation

Playbook
    ↓
Repeatable / structured automation
```

---

## YAML Structure and Indentation

Because playbooks are YAML files, **indentation is significant**.

```yaml
- name: Configure web server
  hosts: webservers

  tasks:
    - name: Install Apache
      ansible.builtin.yum:
        name: httpd
        state: present
```

The hierarchy is:

```text
Play
 ├── name
 ├── hosts
 └── tasks
      └── task
           ├── name
           └── module
                ├── name
                └── state
```

Incorrect indentation can change the YAML structure or cause the playbook to fail.

Use consistent spaces rather than tabs.

---

## Key Takeaways

- An **Ansible playbook** is a YAML file containing one or more plays.
- A **play** defines which hosts should receive a set of tasks.
- A **task** is an individual unit of work.
- A **module** performs the actual operation for a task.
- The `hosts` parameter is defined at the play level.
- A playbook can contain multiple plays targeting different host groups.
- Playbooks make automation repeatable, structured and version-controllable.
- `ansible-playbook` executes playbooks.
- `--syntax-check` checks syntax before execution.
- `--list-hosts` shows targeted hosts.
- `--list-tasks` shows tasks without executing them.
- YAML indentation determines the structure of the playbook.

## Basic Mental Model

```text
Inventory
   ↓
Which machines?

Playbook
   ↓
What should happen?

Play
   ↓
Which hosts receive these tasks?

Task
   ↓
What individual action should happen?

Module
   ↓
How is that action performed?
```

