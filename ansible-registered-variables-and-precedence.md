# Ansible Registered Variables and Variable Precedence

## Registered Variables

Ansible can capture the output and result of a task and store it in a variable using the `register` directive.

This is useful when the result of one task needs to be used by a later task.

For example:

```yaml
---
- name: Demonstrate registered variables
  hosts: localhost
  gather_facts: false

  tasks:
    - name: Read hosts file
      ansible.builtin.command: cat /etc/hosts
      register: result

    - name: Display command result
      ansible.builtin.debug:
        var: result
```

### Note : This .md file is still in progress
