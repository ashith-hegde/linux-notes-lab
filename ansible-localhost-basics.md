# Ansible localhost lab

## Environment

- WSL2 Ubuntu
- Ansible Core 2.20.1
- Python 3.14

## Inventory

```ini
[local]
localhost ansible_connection=local
```

Purpose: `ansible_connection=local` tells Ansible to execute tasks directly on the local machine instead of using SSH.

## Validation

```bash
ansible local -i inventory.ini -m ping
```

Result: The ping module returned `pong`, confirming successful execution on localhost.

## Ad-hoc command practice

### Hostname

```bash
ansible local -i inventory.ini -m command -a "hostname"
```

Purpose: Execute the `hostname` command on the target host.

### Uptime

```bash
ansible local -i inventory.ini -m command -a "uptime"
```

Purpose: Check system uptime and load averages.

### Disk usage

```bash
ansible local -i inventory.ini -m command -a "df -h /"
```

Purpose: Check filesystem usage for the root filesystem.

## Observation

The `command` module executes a command on the target host and returns its output. Unlike the `ping` module, it does not perform a connectivity test; it runs the specified command.

---

# First Playbook

## Playbook file

File: `ping.yml`

```yaml
---
- name: Test localhost
  hosts: local
  gather_facts: false

  tasks:
    - name: Ping localhost
      ansible.builtin.ping:
```

## Run command

```bash
ansible-playbook -i inventory.ini ping.yml
```

## Result

The playbook executed successfully and returned output similar to:

```text
PLAY [Test localhost] *********************************************************

TASK [Ping localhost] *********************************************************
ok: [localhost]

PLAY RECAP ********************************************************************
localhost : ok=1 changed=0 unreachable=0 failed=0
```

## Additional comparison

Ad-hoc command:

```bash
ansible local -i inventory.ini -m command -a "hostname"
```

Playbook command:

```bash
ansible-playbook -i inventory.ini ping.yml
```

### Difference

* **Ad-hoc command:** One-time command execution.
* **Playbook:** Repeatable automation written in YAML.

---

# What I learned

* A playbook is written in YAML.
* YAML indentation is mandatory; spaces define the structure.
* `hosts` defines the target group from the inventory.
* `tasks` contains the actions to execute.
* `ansible.builtin.ping` tests Ansible connectivity and module execution.
* Playbooks are preferred when tasks need to be repeatable, version-controlled, and documented.
* Ad-hoc commands are useful for quick checks, while playbooks are better for automation workflows.

