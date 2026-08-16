# Ansible Variable Scopes and Magic Variables

## Variable Scopes

Variable scope describes the **accessibility or visibility of a variable** — in other words, where a variable can be used during Ansible execution.

Scope and precedence are different concepts:

- **Scope:** Where can the variable be accessed?
- **Precedence:** If multiple applicable variables have the same name, which value wins?

---

## Host Scope

A variable associated with a host is available when a play is being executed for that host.

Host-related variables can come from several places, including:

- Inventory host variables
- `host_vars`
- Group variables inherited by the host
- Facts
- Registered variables
- Other applicable variable sources

For example:

```ini
[webservers]
web1
web2
```

Suppose the group has:

```yaml
# group_vars/webservers.yml

dns_server: 10.1.1.1
```

Both `web1` and `web2` can access `dns_server` because they belong to the `webservers` group.

Conceptually:

```text
webservers
    |
    +-- web1
    |     dns_server = 10.1.1.1
    |
    +-- web2
          dns_server = 10.1.1.1
```

If `web1` has its own host-specific value:

```yaml
# host_vars/web1.yml

dns_server: 10.2.2.2
```

then:

```text
web1 -> dns_server = 10.2.2.2
web2 -> dns_server = 10.1.1.1
```

The host-specific value takes precedence over the group value for `web1`.

### Important concept

Although variables can originate from different places, Ansible ultimately needs to resolve the variables applicable to each host during playbook execution.

Therefore, it is useful to think of the final execution context as being **host-specific**.

---

# Play Scope

A variable defined inside a play is available to tasks within that play.

Example:

```yaml
---
- name: Configure web servers
  hosts: webservers

  vars:
    dns_server: 10.1.1.1

  tasks:
    - name: Display DNS server
      ansible.builtin.debug:
        msg: "DNS server is {{ dns_server }}"
```

Here, `dns_server` is defined at the play level.

It is available to tasks in this play.

If another play defines a different value, that play has its own play-level variable.

Example:

```yaml
---
- name: Configure web servers
  hosts: webservers

  vars:
    environment: production

  tasks:
    - name: Display environment
      ansible.builtin.debug:
        msg: "Environment: {{ environment }}"

- name: Configure development servers
  hosts: development

  vars:
    environment: development

  tasks:
    - name: Display environment
      ansible.builtin.debug:
        msg: "Environment: {{ environment }}"
```

The two plays have different play-level values.

---

# Global / Broadly Available Variables

Some variables can be supplied in a way that makes them available across the relevant playbook execution.

A common example is an **extra variable** supplied from the command line:

```bash
ansible-playbook playbook.yml -e "environment=production"
```

The variable can then be referenced using:

```yaml
{{ environment }}
```

Extra variables also have very high precedence.

### Important distinction

It is useful to think of extra variables as broadly available input to the playbook, but **"global scope" is not a complete description of Ansible's variable model**.

Ansible has multiple variable sources and scopes, so always consider both:

1. Where the variable comes from.
2. Which hosts/plays it applies to.

---

# Magic Variables

Ansible provides special variables called **magic variables**.

Magic variables are variables that Ansible automatically provides during playbook execution.

They allow a playbook to obtain information about:

- The current host
- Other hosts
- Groups
- The inventory
- The current play
- Other execution-related information

Magic variables are particularly useful when automation needs information about the infrastructure itself.

---

## `hostvars`

`hostvars` allows a playbook to access variables belonging to another host.

For example, suppose the inventory contains:

```ini
[webservers]
web1
web2
```

and `web2` has:

```yaml
# host_vars/web2.yml

dns_server: 10.1.1.1
```

A task running for `web1` can access the variable belonging to `web2`:

```yaml
- name: Display web2 DNS server
  ansible.builtin.debug:
    msg: "{{ hostvars['web2']['dns_server'] }}"
```

This produces:

```text
10.1.1.1
```

### Alternative syntax

The following form is also commonly used:

```yaml
{{ hostvars['web2'].dns_server }}
```

Both expressions access the `dns_server` variable associated with `web2`.

---

## Why `hostvars` is useful

Normally, a host's variables are associated with that host.

For example:

```text
web1
  |
  +-- dns_server = 10.1.1.1

web2
  |
  +-- dns_server = 10.2.2.2
```

A task running for `web1` can use:

```yaml
{{ dns_server }}
```

to access its applicable value.

But if it needs the value belonging to `web2`, it can use:

```yaml
{{ hostvars['web2']['dns_server'] }}
```

This makes cross-host automation possible.

---

## Accessing Facts Through `hostvars`

`hostvars` can also be used to access facts gathered for another host, provided those facts are available.

For example:

```yaml
{{ hostvars['web2']['ansible_facts']['hostname'] }}
```

This can be useful when one host needs information about another host.

However, remember that the required facts must actually have been gathered or otherwise made available.

---

# `groups`

The `groups` magic variable provides access to the hosts belonging to inventory groups.

Suppose the inventory contains:

```ini
[webservers]
web1
web2

[dbservers]
db1
db2
```

A playbook can access the hosts in the `webservers` group using:

```yaml
{{ groups['webservers'] }}
```

Conceptually, this represents:

```text
web1
web2
```

### Example

```yaml
- name: Display web servers
  ansible.builtin.debug:
    msg: "{{ groups['webservers'] }}"
```

This is useful when a playbook needs to know which hosts belong to a particular group.

---

## Example: Referencing a Specific Host in a Group

You can also access a particular item from the group list:

```yaml
{{ groups['webservers'][0] }}
```

If the group contains:

```text
web1
web2
```

then the first item would be:

```text
web1
```

Remember that lists are zero-indexed:

```text
[0] -> first item
[1] -> second item
[2] -> third item
```

---

# `group_names`

`group_names` contains the names of the inventory groups to which the **current host** belongs.

For example:

```ini
[webservers]
web1
web2

[production]
web1
web2
```

For `web1`, the `group_names` value would include:

```text
webservers
production
```

A playbook can display it using:

```yaml
- name: Display groups for current host
  ansible.builtin.debug:
    var: group_names
```

This is useful when a host belongs to multiple groups and the playbook needs to determine its group membership.

---

# `inventory_hostname`

`inventory_hostname` contains the name Ansible uses for the current host in the inventory.

For example:

```ini
web1 ansible_host=server1.company.com
```

Ansible has two different names here:

```text
inventory_hostname = web1
ansible_host       = server1.company.com
```

Therefore:

```yaml
{{ inventory_hostname }}
```

would return:

```text
web1
```

It does **not necessarily represent the actual hostname configured on the operating system**.

This distinction is important when aliases are used in the inventory.

---

## Example

Inventory:

```ini
[webservers]
web1 ansible_host=192.168.1.101
web2 ansible_host=192.168.1.102
```

For `web1`:

```yaml
{{ inventory_hostname }}
```

returns:

```text
web1
```

while:

```yaml
{{ ansible_host }}
```

refers to:

```text
192.168.1.101
```

This demonstrates why the inventory name and the actual connection address should not be confused.

---

# Common Magic Variables

Some useful magic variables include:

| Magic variable | Purpose |
|---|---|
| `hostvars` | Access variables/facts associated with hosts |
| `groups` | Access hosts belonging to inventory groups |
| `group_names` | Groups to which the current host belongs |
| `inventory_hostname` | Current host's inventory name |
| `inventory_hostname_short` | Short form of the inventory hostname |
| `ansible_play_hosts` | Hosts currently targeted by the play |
| `ansible_play_batch` | Hosts in the current batch |

These are only a subset of the available magic variables.

---

# Scope vs Magic Variables

These concepts should not be confused.

### Variable scope

Answers:

> "Where is this variable available?"

Example:

```yaml
dns_server: 10.1.1.1
```

### Magic variables

Answer questions such as:

> "What hosts are in this group?"

or:

> "What variables does another host have?"

For example:

```yaml
{{ groups['webservers'] }}
```

and:

```yaml
{{ hostvars['web2']['dns_server'] }}
```

Magic variables provide Ansible's own information about the environment and execution context.

---

# Practical Example

The following example combines several concepts:

```yaml
---
- name: Demonstrate Ansible magic variables
  hosts: webservers
  gather_facts: false

  tasks:
    - name: Display current inventory hostname
      ansible.builtin.debug:
        msg: "Current host: {{ inventory_hostname }}"

    - name: Display current host groups
      ansible.builtin.debug:
        msg: "Groups: {{ group_names }}"

    - name: Display all web servers
      ansible.builtin.debug:
        msg: "Web servers: {{ groups['webservers'] }}"

    - name: Display another host's variable
      ansible.builtin.debug:
        msg: "web2 DNS server: {{ hostvars['web2']['dns_server'] }}"
```

This demonstrates:

```text
inventory_hostname
        |
        +--> Who am I in the inventory?

group_names
        |
        +--> Which groups do I belong to?

groups
        |
        +--> Which hosts belong to this group?

hostvars
        |
        +--> What variables belong to another host?
```

---

# Important Takeaways

- **Variable scope** describes where a variable can be accessed.
- Host-related variables are associated with individual hosts and their execution context.
- Play variables are defined within a play and are available to tasks in that play.
- Extra variables can be supplied from the command line and are broadly available to the playbook execution.
- Scope and precedence are different concepts.
- **Precedence** determines which value wins when multiple applicable variables have the same name.
- **Magic variables** are automatically provided by Ansible.
- `hostvars` allows access to variables and facts associated with other hosts.
- `groups` provides access to hosts belonging to inventory groups.
- `group_names` shows the inventory groups to which the current host belongs.
- `inventory_hostname` identifies the current host using its inventory name.
- An inventory alias can therefore be different from the actual hostname or IP address used for the connection.
- Magic variables are extremely useful for dynamic and multi-host automation.

---

# Official Documentation

The examples above cover only the most commonly useful magic variables.

Ansible provides many more, and their exact behavior is worth understanding before using them in complex automation.

**For the complete list and detailed behavior, visit the official Ansible documentation:**

https://docs.ansible.com/projects/ansible/latest/playbook_guide/playbooks_vars_facts.html

Pay particular attention to the sections covering:

- Using variables
- Variable scope
- Magic variables
- `hostvars`
- `groups`
- `group_names`
- Inventory variables
- Facts

For variable precedence, see:

https://docs.ansible.com/projects/ansible/latest/reference_appendices/general_precedence.html

