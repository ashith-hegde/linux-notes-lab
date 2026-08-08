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
