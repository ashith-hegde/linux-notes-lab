# Ansible Variables

## What are variables?

Variables in Ansible are used to store values that may change between hosts, environments, or executions.

Just like in any scripting or programming language, variables help avoid hardcoding values inside playbooks.

Variables are commonly used to store:

* host-specific information,
* connection settings,
* ports,
* usernames,
* passwords,
* and configuration values.

---

# Inventory Variables

Values can be defined directly in the inventory file.

```ini
web1 ansible_host=server1.company.com ansible_connection=ssh ansible_ssh_pass=P@ssW

db1 ansible_host=server2.company.com ansible_connection=winrm ansible_password=P@ss

web2 ansible_host=server3.company.com ansible_connection=ssh ansible_ssh_pass=P@ssW
```

In this example:

* `ansible_host` → actual hostname or IP address,
* `ansible_connection` → connection method,
* `ansible_ssh_pass` → SSH password for Linux hosts,
* `ansible_password` → generic connection password (commonly used with WinRM).

These are examples of inventory variables.

---

# Variables Inside a Playbook

## Playbook without variables

```yaml
---
- name: Add DNS server to resolv.conf
  hosts: localhost

  tasks:
    - name: Update resolv.conf
      ansible.builtin.lineinfile:
        path: /etc/resolv.conf
        line: "nameserver 10.1.250.10"
```

The DNS server is hardcoded.

---

## Playbook using a variable

```yaml
---
- name: Add DNS server to resolv.conf
  hosts: localhost

  vars:
    dns_server: 10.1.250.10

  tasks:
    - name: Update resolv.conf
      ansible.builtin.lineinfile:
        path: /etc/resolv.conf
        line: "nameserver {{ dns_server }}"
```

The value of `dns_server` can now be changed in one place.

---

# Variable Files

Variables can also be stored in a dedicated file.

## Example: `variables.yml`

```yaml
variable1: value1
variable2: value2
```

Load the file in a playbook:

```yaml
vars_files:
  - variables.yml
```

All variables defined in that file become available to the playbook.

---

# Why Use Variables?

Imagine a long firewall playbook containing many hardcoded values.

If another engineer wants to reuse the playbook, they would need to edit the playbook itself.

By moving changing values into variables, only the variable file needs to be modified.

This improves:

* reusability,
* maintainability,
* readability,
* and environment customization.

---

# Example: Firewall Configuration

## Variable file (`web.yml`)

```yaml
http_port: 8081
snmp_port: 161
internal_ip_range: 192.0.2.0
```

## Playbook

```yaml
---
- name: Set firewall configurations
  hosts: web
  become: true

  vars_files:
    - web.yml

  tasks:
    - name: Enable HTTPS service
      ansible.posix.firewalld:
        service: https
        permanent: true
        state: enabled

    - name: Enable HTTP port
      ansible.posix.firewalld:
        port: "{{ http_port }}/tcp"
        permanent: true
        state: enabled

    - name: Enable SNMP port
      ansible.posix.firewalld:
        port: "{{ snmp_port }}/udp"
        permanent: true
        state: enabled

    - name: Allow internal network
      ansible.posix.firewalld:
        source: "{{ internal_ip_range }}/24"
        zone: internal
        state: enabled
        permanent: true
```

All values come from the variable file rather than being hardcoded.

---

# Jinja2 Templating (`{{ }}`)

Ansible uses **Jinja2** syntax for variable substitution.

## Correct

```yaml
source: "{{ internal_ip_range }}"
```

```yaml
msg: "Port is {{ http_port }}"
```

## Wrong

```yaml
source: {{ internal_ip_range }}
```

### Important note

If the value starts with a variable expression, quote the whole value.

---

# Types of Variables

## String Variables

String variables store text values.

```yaml
username: "admin"
environment: "production"
```

String variables can be defined in a playbook, inventory, or passed as command-line arguments.

---

## Number Variables

Number variables store integer or floating-point values.

```yaml
max_connections: 100
timeout_seconds: 2.5
```

They can be used in mathematical operations and comparisons.

---

## Boolean Variables

Boolean variables store `true` or `false`.

```yaml
debug_mode: true
maintenance_mode: false
```

They are often used in conditional statements.

### Common truthy values

* `true`
* `yes`
* `on`
* `1`

### Common falsy values

* `false`
* `no`
* `off`
* `0`

---

## List Variables

List variables store an ordered collection of values.

```yaml
packages:
  - nginx
  - postgresql
  - git
```

### Access values

Entire list:

```yaml
{{ packages }}
```

First item:

```yaml
{{ packages[0] }}
```

### Example playbook

```yaml
---
- name: Install packages
  hosts: localhost

  vars:
    packages:
      - nginx
      - git

  tasks:
    - name: Display all packages
      debug:
        var: packages

    - name: Display the first package
      debug:
        msg: "The first package is {{ packages[0] }}"

    - name: Simulate package installation
      debug:
        msg: "Installing package {{ item }}"
      loop: "{{ packages }}"
```

---

## Dictionary Variables

Dictionary variables store key-value pairs.

```yaml
user:
  name: admin
  password: secret
```

### Access values

Entire dictionary:

```yaml
{{ user }}
```

Specific key:

```yaml
{{ user.name }}
```

### Example playbook

```yaml
---
- name: Access dictionary variable
  hosts: localhost

  vars:
    user:
      name: admin
      password: secret

  tasks:
    - name: Display the entire dictionary
      debug:
        var: user

    - name: Display specific values
      debug:
        msg: "Username: {{ user.name }}, Password: {{ user.password }}"
```

Dictionary entries are referenced using their keys.

---

# Key Takeaways

* Variables store values that may change.
* Inventory variables define host-specific settings.
* `vars:` defines variables inside a playbook.
* `vars_files:` loads variables from external files.
* Jinja2 syntax uses `{{ variable_name }}`.
* Lists use indexes such as `packages[0]`.
* Dictionaries use keys such as `user.name`.
* Variables make playbooks reusable and easier to maintain.

