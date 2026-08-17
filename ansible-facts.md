# Ansible Facts

## What are Ansible Facts?

When Ansible connects to a managed host during playbook execution, it can automatically collect information about that host.

This information is called **Ansible facts**.

Facts describe the current state and characteristics of the managed host, such as:

- Operating system and OS version
- System architecture
- Processor/CPU information
- Hostname and FQDN
- Network interfaces
- IP addresses
- MAC addresses
- Disks and storage devices
- Filesystems and mount points
- Available disk space
- Memory information
- Date and time
- Other system information and settings

Facts are useful when a playbook needs to make decisions based on the characteristics of the target host.

---

## How does Ansible gather facts?

Ansible uses the **`ansible.builtin.setup` module** to gather facts.

The `setup` module is normally run automatically at the beginning of a play when fact gathering is enabled.

For example:

```yaml
---
- name: Gather information about hosts
  hosts: all

  tasks:
    - name: Display hostname
      ansible.builtin.debug:
        var: ansible_hostname
```

There is no explicit `setup` task in this playbook, but Ansible gathers facts automatically before executing the tasks.

Conceptually:

```text
Playbook starts
      |
      v
Connect to managed host
      |
      v
Gather facts using setup module
      |
      v
Run playbook tasks
```

---

## Viewing Facts Manually

The `setup` module can also be executed explicitly.

For example:

```bash
ansible localhost -m ansible.builtin.setup
```

This returns a large amount of information about the host.

To view a particular fact, you can use the `filter` option.

Example:

```bash
ansible localhost -m ansible.builtin.setup -a "filter=ansible_hostname"
```

Another example:

```bash
ansible localhost -m ansible.builtin.setup -a "filter=ansible_distribution*"
```

This can be useful when exploring which facts are available before using them in a playbook.

---

## Facts and `ansible_facts`

Ansible exposes gathered information through facts.

Modern Ansible commonly represents the structured fact data under:

```yaml
ansible_facts
```

For example:

```yaml
ansible_facts:
  hostname: server01
  distribution: Ubuntu
```

Individual fact variables are also commonly available for convenient access, depending on the fact and Ansible version.

For example:

```yaml
{{ ansible_hostname }}
```

or:

```yaml
{{ ansible_facts['hostname'] }}
```

The second form explicitly accesses the fact through the `ansible_facts` dictionary.

---

## Example: Using Facts in a Playbook

Facts become particularly useful when a playbook needs to behave differently depending on the target system.

Example:

```yaml
---
- name: Display system information
  hosts: all

  tasks:
    - name: Display operating system
      ansible.builtin.debug:
        msg: "OS: {{ ansible_facts['distribution'] }}"

    - name: Display hostname
      ansible.builtin.debug:
        msg: "Hostname: {{ ansible_facts['hostname'] }}"

    - name: Display CPU architecture
      ansible.builtin.debug:
        msg: "Architecture: {{ ansible_facts['architecture'] }}"
```

The same playbook can therefore obtain information from each managed host dynamically instead of hard-coding the values.

---

## Disabling Fact Gathering

Fact gathering is enabled by default.

If a playbook does not depend on facts, fact gathering can be disabled:

```yaml
---
- name: Example without fact gathering
  hosts: all
  gather_facts: false

  tasks:
    - name: Display message
      ansible.builtin.debug:
        msg: "Facts were not gathered for this play."
```

You can also use:

```yaml
gather_facts: no
```

although `false` is generally clearer in YAML.

### Why disable fact gathering?

Fact gathering requires Ansible to collect information from the managed hosts before running the tasks.

For a simple playbook that does not use facts, disabling it can:

- Reduce unnecessary work
- Reduce execution time
- Avoid gathering information that the playbook does not need

For example, a simple connectivity test does not normally require facts:

```yaml
---
- name: Test connectivity
  hosts: all
  gather_facts: false

  tasks:
    - name: Ping hosts
      ansible.builtin.ping:
```

---

## Fact Gathering and Configuration

Fact gathering is enabled by default, but its behavior can also be influenced through Ansible configuration.

For example, configuration can be controlled through `ansible.cfg`.

If a setting is specified both in the configuration and explicitly in a playbook, the **more specific play-level setting takes precedence**.

For example, if configuration disables fact gathering:

```ini
[defaults]
gathering = explicit
```

and a play explicitly enables it:

```yaml
---
- name: Example
  hosts: all
  gather_facts: true

  tasks:
    - name: Display hostname
      ansible.builtin.debug:
        var: ansible_hostname
```

the explicit play setting determines the behavior for that play.

### Important clarification

`gathering` in `ansible.cfg` and `gather_facts` in a play are related but are **not simply two versions of the same setting**.

- `gather_facts` controls whether a particular play gathers facts.
- `gathering` is a configuration setting that controls fact-gathering behavior/caching strategy.

For normal learning and most playbooks, focus first on:

```yaml
gather_facts: true
```

or:

```yaml
gather_facts: false
```

---

## Key Takeaways

- **Ansible facts** are information collected about managed hosts.
- Facts include OS, CPU, memory, networking, storage, hostname and other system information.
- Ansible normally gathers facts automatically at the beginning of a play.
- The `ansible.builtin.setup` module is responsible for fact gathering.
- Facts can be inspected directly using the `setup` module.
- Fact data is available through `ansible_facts` and commonly exposed individual fact variables.
- Facts allow playbooks to make decisions dynamically based on the target system.
- Fact gathering can be disabled with:

```yaml
gather_facts: false
```

- Disabling fact gathering is useful when the playbook does not require system facts.

