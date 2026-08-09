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

## Validation

```bash
ansible local -i inventory.ini -m ping
```

Result: `pong` was returned successfully.

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

