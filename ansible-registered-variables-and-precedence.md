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

### How it works

The first task uses:

```yaml
register: result
```

This stores the result of the command in a variable called `result`.

The next task can then access that variable.

```text
Task 1
  |
  | Run command
  v
register: result
  |
  v
Task 2
  |
  | Use result
  v
result.stdout / result.rc / etc.
```

\---

## Accessing Parts of a Registered Variable

A registered variable usually contains more information than just the command output.

For example:

```yaml
result.stdout
```

contains the standard output of the command.

Other commonly useful attributes include:

```yaml
result.stdout
result.stderr
result.rc
result.changed
result.failed
```

### Common attributes

|Attribute|Meaning|
|-|-|
|`stdout`|Standard output produced by the command|
|`stderr`|Standard error output|
|`rc`|Return code of the command|
|`changed`|Whether Ansible considered the task changed|
|`failed`|Whether the task failed|

The exact fields available depend on the module and task that produced the result.

### Example

```yaml
---
- name: Demonstrate registered variable fields
  hosts: localhost
  gather\_facts: false

  tasks:
    - name: Check hostname
      ansible.builtin.command: hostname
      register: hostname\_result

    - name: Display hostname
      ansible.builtin.debug:
        msg: "{{ hostname\_result.stdout }}"

    - name: Display return code
      ansible.builtin.debug:
        msg: "Return code: {{ hostname\_result.rc }}"
```

\---

## Using Registered Variables in Later Tasks

Registered variables are particularly useful when the result of one task determines what another task should do.

Example:

```yaml
---
- name: Check nginx status
  hosts: webservers
  gather\_facts: false

  tasks:
    - name: Check whether nginx is running
      ansible.builtin.command: systemctl is-active nginx
      register: nginx\_status
      changed\_when: false

    - name: Display nginx status
      ansible.builtin.debug:
        msg: "Nginx status: {{ nginx\_status.stdout }}"
```

The registered variable can also be used in a condition:

```yaml
- name: Check whether nginx is running
  ansible.builtin.command: systemctl is-active nginx
  register: nginx\_status
  changed\_when: false

- name: Report if nginx is not running
  ansible.builtin.debug:
    msg: "Nginx is not running"
  when: nginx\_status.stdout != "active"
```

This allows an Ansible playbook to make decisions based on the result of previous tasks.

\---

## Registered Variable Scope

A variable created using `register` is associated with the host on which the task runs.

It is available for the remainder of the current playbook execution.

Example:

```yaml
- name: Get hostname
  ansible.builtin.command: hostname
  register: hostname\_result

- name: Display hostname
  ansible.builtin.debug:
    var: hostname\_result
```

The second task can use `hostname\_result` because it runs later during the same playbook execution.

### Important

Registered variables are stored in memory.

They are **not automatically persisted for future playbook runs**.

If the playbook is executed again, the registered variable is created again during that execution.

\---

## Viewing Task Results with Verbosity

The `debug` module is useful when you specifically want to inspect a variable.

Ansible also provides verbosity options that can help when troubleshooting:

```bash
ansible-playbook playbook.yml -v
```

Higher verbosity levels provide more execution information:

```bash
ansible-playbook playbook.yml -vv
ansible-playbook playbook.yml -vvv
```

Use verbosity when troubleshooting rather than relying on it as a replacement for understanding registered variables.

\---

# Variable Precedence

## What Is Variable Precedence?

Ansible allows variables to be defined in many different places.

For example:

* Inventory group variables
* Inventory host variables
* Play variables
* Variable files
* Registered variables
* Extra variables passed using `-e`

Sometimes the same variable name may be defined in more than one location.

Ansible then needs to determine which value should be used.

This is called **variable precedence**.

> When the same variable is defined in multiple applicable locations, the value with higher precedence overrides the lower-precedence value.

\---

## Simple Example

Suppose a group variable defines:

```yaml
dns\_server: 10.1.1.1
```

A host-specific variable could define:

```yaml
dns\_server: 10.2.2.2
```

The host-specific value takes precedence over the applicable group value.

Conceptually:

```text
Group variable
dns\_server = 10.1.1.1
        |
        v
Host variable
dns\_server = 10.2.2.2
        |
        v
Value used by that host
dns\_server = 10.2.2.2
```

This is useful because a common value can be defined for an entire group while individual hosts can override it when necessary.

\---

## How Ansible Associates Group Variables with Hosts

Suppose the inventory contains:

```ini
\[webservers]
web1
web2
```

and the group has:

```yaml
dns\_server: 10.1.1.1
```

Conceptually:

```text
webservers
    |
    +-- web1
    |     dns\_server = 10.1.1.1
    |
    +-- web2
          dns\_server = 10.1.1.1
```

If `web1` has its own host variable:

```yaml
dns\_server: 10.2.2.2
```

then:

```text
webservers
    |
    +-- web1
    |     dns\_server = 10.2.2.2
    |
    +-- web2
          dns\_server = 10.1.1.1
```

The host-specific value overrides the group value for `web1`.

\---

## Simplified Variable Precedence Model

For the concepts covered so far, a useful simplified model is:

```text
Lower precedence
      |
      v
Group variables
      |
      v
Host variables
      |
      v
Play variables
      |
      v
Extra variables (-e)
      |
      v
Higher precedence
```

For example:

```text
Group:
dns\_server = 10.1.1.1

Host:
dns\_server = 10.2.2.2

Play:
dns\_server = 10.3.3.3

Extra vars:
dns\_server = 10.5.5.6
```

The final value would be:

```text
dns\_server = 10.5.5.6
```

because the extra variable has higher precedence.

### Important clarification

This is a **simplified learning model**, not the complete Ansible precedence hierarchy.

Ansible has many additional variable sources, including:

* Role defaults
* `group\_vars`
* `host\_vars`
* Facts
* `set\_fact`
* Play variables
* `vars\_files`
* Block variables
* Task variables
* `include\_vars`
* Registered variables
* Role parameters
* Include parameters
* Extra variables

The exact precedence order matters when building complex Ansible projects.

\---

# Extra Variables

Extra variables can be supplied when running a playbook using:

```bash
-e
```

or:

```bash
--extra-vars
```

Example:

```bash
ansible-playbook playbook.yml --extra-vars "dns\_server=10.5.5.6"
```

Short form:

```bash
ansible-playbook playbook.yml -e "dns\_server=10.5.5.6"
```

### Correct syntax

Use:

```text
dns\_server=10.5.5.6
```

rather than:

```text
dns\_server = 10.5.5.6
```

\---

## Example of Extra Variable Overriding Other Variables

Suppose the playbook contains:

```yaml
---
- name: Configure DNS
  hosts: webservers

  vars:
    dns\_server: 10.1.1.1

  tasks:
    - name: Display DNS server
      ansible.builtin.debug:
        msg: "DNS server is {{ dns\_server }}"
```

Without extra variables, the play uses:

```text
10.1.1.1
```

Run:

```bash
ansible-playbook playbook.yml -e "dns\_server=10.5.5.6"
```

The output uses:

```text
10.5.5.6
```

because extra variables have the highest precedence among Ansible variables.

\---

## Extra Variables and Data Types

When using the simple:

```bash
key=value
```

format, values are commonly passed as strings.

For example:

```bash
ansible-playbook playbook.yml -e "max\_connections=100"
```

For structured values such as integers, booleans, lists, or dictionaries, JSON/YAML syntax can be used.

Example:

```bash
ansible-playbook playbook.yml \\
  --extra-vars '{"max\_connections": 100, "debug\_mode": true}'
```

This is useful when passing structured configuration to a playbook.

\---

# Registered Variables vs Normal Variables

These concepts are related but different.

## Normal Variable

A normal variable is explicitly defined:

```yaml
vars:
  dns\_server: 10.1.1.1
```

The value is known when the variable is defined.

## Registered Variable

A registered variable gets its value from the result of a task:

```yaml
- name: Get hostname
  ansible.builtin.command: hostname
  register: hostname\_result
```

The value comes from task execution.

Conceptually:

```text
Normal variable
    |
    +-- Value is explicitly defined


Registered variable
    |
    +-- Value is produced by a task
```

\---

# Example Combining Variables and Registered Variables

The two concepts can be used together:

```yaml
---
- name: Demonstrate variables
  hosts: localhost
  gather\_facts: false

  vars:
    command\_to\_run: hostname

  tasks:
    - name: Run command
      ansible.builtin.command: "{{ command\_to\_run }}"
      register: command\_result

    - name: Display command output
      ansible.builtin.debug:
        msg: "The hostname is {{ command\_result.stdout }}"
```

Here:

```text
command\_to\_run
    |
    | Normal variable
    v
hostname command
    |
    | register
    v
command\_result
    |
    | Registered variable
    v
command\_result.stdout
```

This demonstrates an important Ansible pattern:

> \*\*Variables provide input to tasks, while registered variables can capture task output for later tasks.\*\*

\---

# Key Takeaways

* Ansible can capture the result of a task using `register`.
* A registered variable is created from the result of an Ansible task.
* `register` allows information from one task to be used by later tasks.
* Registered variables commonly contain fields such as `stdout`, `stderr`, `rc`, `changed`, and `failed`.
* The exact fields available depend on the module that produced the result.
* Registered variables are associated with the host on which the task runs.
* Registered variables are stored in memory and are not automatically persisted between playbook runs.
* The `debug` module can be used to inspect variables.
* `-v`, `-vv`, and `-vvv` can provide additional task execution information.
* Variable precedence determines which value Ansible uses when the same variable is defined in multiple applicable locations.
* Host-specific variables can override applicable group variables.
* Extra variables supplied using `-e` or `--extra-vars` have the highest precedence among Ansible variables.
* The simplified `group → host → play → extra vars` model is useful for learning, but it is **not the complete precedence hierarchy**.
* Variables can provide input to tasks, while registered variables can capture task output.
* Registered variables are particularly useful when later tasks need to make decisions based on earlier task results.

\---

# Official Documentation

The simplified precedence model covered in this lesson is only a starting point.

Ansible has a much larger variable-precedence hierarchy with many additional sources and special cases.

**Visit the official documentation to get the full picture before relying on precedence behavior in complex projects.**

### Variable Precedence

https://docs.ansible.com/projects/ansible/latest/reference\_appendices/general\_precedence.html

### Using Variables

https://docs.ansible.com/projects/ansible/latest/playbook\_guide/playbooks\_variables.html

These documents cover:

* Inventory variables
* `group\_vars`
* `host\_vars`
* Play variables
* Task variables
* Role variables
* `set\_fact`
* Registered variables
* Extra variables
* Variable merging
* Variable precedence

\---

# Further Reading

* [Ansible — Using Variables](https://docs.ansible.com/projects/ansible/latest/playbook_guide/playbooks_variables.html)
* [Ansible — Variable Precedence](https://docs.ansible.com/projects/ansible/latest/reference_appendices/general_precedence.html)


