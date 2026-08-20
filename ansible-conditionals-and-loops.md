# Ansible Conditionals and Loops

## Ansible Conditionals

Conditionals allow Ansible to control whether a task is executed based on a specific condition.

The main keyword used for task conditionals is:

```yaml
when:
```

A task runs only when its `when` condition evaluates to `true`.

### Example: Check the OS Family

```yaml
- name: Install a package only on Red Hat systems
  ansible.builtin.dnf:
    name: httpd
    state: present
  when: ansible_os_family == "RedHat"
```

Here:

- `ansible_os_family` is an Ansible fact.
- Ansible gathers this information about the target host.
- The task runs only if the OS family is `RedHat`.

> Note: `ansible_os_family` is populated from Ansible facts, so fact gathering must be available if the playbook depends on this fact.

---

## OR Conditions

Use OR when at least one condition must be true.

```yaml
- name: Run task on Red Hat or Debian systems
  ansible.builtin.debug:
    msg: "This is a supported operating system family"
  when: ansible_os_family == "RedHat" or ansible_os_family == "Debian"
```

The task runs when either condition is true.

---

## AND Conditions

Use AND when all specified conditions must be true.

```yaml
- name: Run task only on Red Hat version 9 systems
  ansible.builtin.debug:
    msg: "This task runs on Red Hat 9"
  when:
    - ansible_os_family == "RedHat"
    - ansible_distribution_major_version == "9"
```

Both conditions must be true.

This can also be written explicitly:

```yaml
when: ansible_os_family == "RedHat" and ansible_distribution_major_version == "9"
```

Using a YAML list is often easier to read for multiple AND conditions.

---

## Conditionals with Loops

Conditionals can be used together with loops. The condition can be evaluated for each loop item.

```yaml
- name: Create selected users
  ansible.builtin.user:
    name: "{{ item.name }}"
    uid: "{{ item.uid }}"
    state: present
  loop:
    - name: joe
      uid: 1010
    - name: jane
      uid: 1011
  when: item.uid > 1010
```

The task processes each user, but only users whose UID is greater than `1010` are created.

---

# Ansible Loops

A loop allows the same task to be executed multiple times using different values.

The main modern loop keyword is:

```yaml
loop:
```

For each iteration, the current value is available through the default loop variable:

```text
item
```

Loops reduce repetition and make playbooks easier to maintain.

## Example: Simple Loop

```yaml
- name: Install multiple packages
  hosts: localhost
  become: true

  tasks:
    - name: Install packages
      ansible.builtin.package:
        name: "{{ item }}"
        state: present
      loop:
        - nginx
        - git
        - curl
```

During execution, `item` becomes:

```text
nginx
git
curl
```

Conceptually, Ansible performs:

```text
Install nginx
Install git
Install curl
```

using one task.

---

## Loop with a List of Dictionaries

A loop can process dictionaries containing multiple related values.

```yaml
- name: Create users
  hosts: localhost
  become: true

  tasks:
    - name: Create users
      ansible.builtin.user:
        name: "{{ item.name }}"
        uid: "{{ item.uid }}"
        state: present
      loop:
        - name: joe
          uid: 1010
        - name: jane
          uid: 1011
```

For the first iteration:

```text
item.name = joe
item.uid = 1010
```

For the second:

```text
item.name = jane
item.uid = 1011
```

### JSON-Style Representation

The same data can also be written inline:

```yaml
loop:
  - { name: joe, uid: 1010 }
  - { name: jane, uid: 1011 }
```

The YAML block format is generally easier to read when the data becomes more complex.

---

# `with_*` Loop Syntax

Before the modern `loop` keyword became the preferred general approach, Ansible commonly used directives beginning with:

```text
with_
```

Examples include:

```text
with_items
with_file
with_url
with_mongodb
```

Example:

```yaml
- name: Display multiple values
  ansible.builtin.debug:
    msg: "{{ item }}"
  with_items:
    - nginx
    - git
    - curl
```

Modern equivalent:

```yaml
- name: Display multiple values
  ansible.builtin.debug:
    msg: "{{ item }}"
  loop:
    - nginx
    - git
    - curl
```

`with_*` syntax is still important to understand because it appears in existing and older Ansible playbooks.

---

# Lookup Plugins and `with_*`

The part after `with_` historically corresponds to a lookup plugin.

Lookup plugins allow Ansible to retrieve data from sources such as:

- Files
- URLs
- Environment variables
- Databases
- Key/value stores
- APIs
- Other services

Examples include:

```text
with_file
with_url
```

Conceptually:

```text
External data source
        ↓
Lookup plugin retrieves data
        ↓
Ansible receives the data
        ↓
Task processes the resulting values
```

Modern Ansible also allows lookup plugins to be used with `lookup()` and `query()` together with `loop`.

> A loop controls repeated task execution, while a lookup plugin can provide data that Ansible uses during execution.

---

# Conditionals and Loops Together

```yaml
- name: Install selected packages
  hosts: localhost
  become: true

  vars:
    packages:
      - name: nginx
        enabled: true
      - name: telnet
        enabled: false
      - name: git
        enabled: true

  tasks:
    - name: Install enabled packages
      ansible.builtin.package:
        name: "{{ item.name }}"
        state: present
      loop: "{{ packages }}"
      when: item.enabled
```

For each package:

1. Ansible processes one item.
2. Ansible evaluates `item.enabled`.
3. The package is installed only when the condition is `true`.

This demonstrates how loops and conditionals can be combined to avoid repeating tasks.

---

# Key Takeaways

- `when` controls whether a task is executed.
- A task runs only when its `when` condition evaluates to true.
- Ansible facts such as `ansible_os_family` can be used in conditions.
- Use `or` when either condition should allow execution.
- Multiple conditions written as a YAML list under `when` are evaluated as AND conditions.
- `loop` executes the same task repeatedly with different values.
- The current loop value is stored in `item` by default.
- Lists can contain simple values or dictionaries.
- Dictionary values can be accessed using expressions such as `item.name`.
- Conditions can be evaluated separately for each item in a loop.
- `with_*` is older loop syntax that is still encountered in existing Ansible content.
- Lookup plugins can retrieve data from external or other sources for use by Ansible.
- For most new general-purpose loops, `loop` is the clearer modern approach.

