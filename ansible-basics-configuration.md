# Ansible Basics - Configuration

Notes from the KodeKloud course: *Ansible for the Absolute Beginner*.

---

## Why Ansible?

Ansible is an automation tool commonly used for:

* Provisioning
* Configuration management
* Continuous delivery
* Application deployment
* Security compliance

---

## Challenges with Script-Based Automation

Traditional shell-script-based automation can become difficult because of:

* Time required to develop and maintain scripts
* Higher coding effort
* Maintenance overhead as environments grow

---

## Advantages of Ansible

Ansible provides:

* Simple automation framework
* Powerful configuration management capabilities
* Agentless architecture

For Linux systems, Ansible commonly uses **SSH** to communicate with managed nodes.

---

## Ansible Configuration Files

The main system-wide configuration file is:

```text
/etc/ansible/ansible.cfg
```

Example configuration:

```ini
[defaults]
inventory = /etc/ansible/hosts
log_path = /var/log/ansible.log
library = /usr/share/my_modules/
roles_path = /etc/ansible/roles
action_plugins = /usr/share/ansible/plugins/action
gathering = implicit
timeout = 10
forks = 5

[inventory]
enable_plugins = host_list, virtualbox, yaml, constructed
```

> These are example settings; actual values may differ between environments.

---

## Project-Level Configuration Override

Instead of modifying the global configuration file, a custom `ansible.cfg` can be placed in the project directory:

```text
project-directory/
├── ansible.cfg
├── inventory.ini
└── playbook.yml
```

This allows project-specific configuration without changing the system-wide settings.

---

## Configuration Precedence

Ansible uses the following order of precedence for configuration files:

1. `ANSIBLE_CONFIG` environment variable
2. `ansible.cfg` in the current directory
3. `~/.ansible.cfg`
4. `/etc/ansible/ansible.cfg`

Higher-precedence configurations override lower-precedence configurations.

---

## Environment Variable Override

Individual settings can be overridden for a single command execution.

Example:

```bash
ANSIBLE_GATHERING=explicit ansible-playbook site.yml
```

This changes the gathering behavior only for that execution.

---

## Finding Environment Variable Names

Many configuration options have corresponding environment variables.

Example:

| Configuration option | Environment variable |
| -------------------- | -------------------- |
| `gathering`          | `ANSIBLE_GATHERING`  |

The exact variable name should be verified using documentation or the `ansible-config` command.

---

## Useful Ansible Configuration Commands

### List all available configuration options

```bash
ansible-config list
```

### View the active configuration file

```bash
ansible-config view
```

### View effective current settings

```bash
ansible-config dump
```

---

## Important Concepts

### Agentless Architecture

Managed Linux nodes do not require an Ansible agent installation. The control node connects using SSH and executes modules remotely.

### Configuration Override Principle

A custom `ansible.cfg` only needs the parameters that should be changed. Unspecified parameters continue to use lower-precedence values.

---

## Key Notes

### What is Ansible?

Ansible is an open-source automation and configuration management tool used for provisioning, configuration, deployment, and operational automation.

### Why is Ansible called agentless?

Because managed nodes do not require an Ansible agent; the control node connects over SSH.

### How can configuration be overridden temporarily?

Using environment variables, for example:

```bash
ANSIBLE_GATHERING=explicit ansible-playbook site.yml
```

### Common Configuration Locations

* System-wide: `/etc/ansible/ansible.cfg`
* User-level: `~/.ansible.cfg`
* Project-level: `./ansible.cfg`

Project-level configuration is commonly used to keep automation projects self-contained.

