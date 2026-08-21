# Ansible Modules

## What Are Ansible Modules?

Ansible modules are reusable units of code that perform specific actions on managed hosts. When a task runs, Ansible uses a module to perform the requested operation.

Modules are broadly organized by functionality, for example:

- System
- Commands
- Files
- Database
- Cloud
- Windows
- More

> Note: Module availability can depend on installed Ansible collections. For the full and current list of modules and their instructions, refer to the official Ansible documentation.

---

## System Modules

System modules perform operating-system and system-level administration tasks.

Examples include:

- `user`
- `group`
- `hostname`
- `iptables`
- `lvg`
- `lvol`
- `make`
- `mount`
- `ping`
- `timezone`
- `systemd`
- `service`

Example:

```yaml
- name: Create a user
  ansible.builtin.user:
    name: devuser
    state: present
```

---

## Command Modules

Command-related modules execute commands or scripts on managed hosts.

Examples include:

- `command`
- `raw`
- `shell`
- `expect`
- `script`

Different modules are appropriate for different situations. For example, `command` runs commands without using a shell, while `shell` uses a shell.

---

## Database Modules

Database modules help automate database administration and configuration.

Depending on the database and collection, they can be used to:

- Create or remove databases
- Manage database users
- Modify configuration
- Perform database-related operations

Examples include modules or collections related to:

- MongoDB
- Microsoft SQL Server
- PostgreSQL
- ProxySQL
- Vertica

---

## Cloud Modules

Cloud-related modules and collections allow Ansible to manage infrastructure resources on platforms such as:

- Amazon Web Services
- Docker
- Google Cloud
- VMware

They can automate infrastructure creation, configuration, and management.

---

## Windows Modules

Ansible can also manage Windows machines using Windows-specific modules.

These can be used for tasks such as:

- Managing Windows services
- Running PowerShell commands
- Managing Windows features
- Managing files and configuration
- Installing software

---

# The `command` Module

The `command` module runs a command on a managed host.

Basic example:

```yaml
- name: Display the current date
  ansible.builtin.command: date
```

Conceptually:

```text
Module: command
Value:  date
```

The module name identifies the action, while the command provides the input.

---

## Free-Form Input

The `command` module accepts free-form input.

Example:

```yaml
ansible.builtin.command: cat /etc/resolv.conf
```

Here:

```text
cat /etc/resolv.conf
```

is the free-form command input. The command itself is provided directly rather than through a parameter such as `name`.

Not every Ansible module supports free-form input.

---

## The `chdir` Parameter

`chdir` changes the working directory before the command is executed.

Without `chdir`:

```yaml
- name: Display resolv.conf
  ansible.builtin.command: cat /etc/resolv.conf
```

With `chdir`:

```yaml
- name: Display resolv.conf
  ansible.builtin.command:
    cmd: cat resolv.conf
    chdir: /etc
```

Conceptually:

```text
Change directory to /etc
        ↓
Run: cat resolv.conf
```

---

## The `creates` Parameter

The `creates` parameter checks whether a path already exists before running a command.

Example:

```yaml
- name: Create a directory if it does not already exist
  ansible.builtin.command:
    cmd: mkdir /folder
    creates: /folder
```

Conceptually:

```text
Does /folder exist?
       │
       ├── Yes → Skip the command
       │
       └── No  → Run mkdir /folder
```

When a dedicated module exists, it is generally preferable to use it.

For example, instead of using `mkdir` through `command`, use the `file` module:

```yaml
- name: Ensure the directory exists
  ansible.builtin.file:
    path: /folder
    state: directory
```

This more clearly describes the desired state and is naturally idempotent.

---

# Parameterized Modules

Some modules require named parameters instead of free-form input.

For example, `copy` copies a file from a source to a destination:

```yaml
- name: Copy the hosts file
  ansible.builtin.copy:
    src: /etc/hosts
    dest: /tmp/hosts
```

Here:

- `src` specifies the source.
- `dest` specifies the destination.

---

# The `script` Module

The `script` module runs a script located on the Ansible controller on managed hosts.

Example:

```yaml
- name: Run a script on managed hosts
  ansible.builtin.script: scripts/check_disk.sh
```

The path refers to the script's location on the **Ansible controller**, not on the managed host.

Ansible transfers the script to the managed host and executes it automatically. Therefore, the script does not need to be manually copied to every server beforehand.

This works whether the play targets one managed host or many hosts.

---

## Passing Arguments to a Script

Arguments can be passed directly to the script:

```yaml
- name: Run script with arguments
  ansible.builtin.script: scripts/check_disk.sh arg1 arg2
```

Conceptually:

```text
Controller script
      ↓
Transferred to managed host
      ↓
Executed with arg1 and arg2
```

---

# The `service` Module

The `service` module manages services on a system.

Common operations include:

- Starting a service
- Stopping a service
- Restarting a service
- Ensuring a service remains in the desired state

Unlike `command`, the `service` module uses named parameters.

The main parameters are:

- `name` — Which service should be managed?
- `state` — What state should the service be in?

Example:

```yaml
- name: Start services in order
  hosts: localhost

  tasks:
    - name: Start the database
      ansible.builtin.service:
        name: postgresql
        state: started
```

---

## One-Line and Dictionary Syntax

The same parameters can also be written on one line:

```yaml
ansible.builtin.service: name=postgresql state=started
```

The dictionary format is generally easier to read:

```yaml
ansible.builtin.service:
  name: postgresql
  state: started
```

---

# Why `started` Instead of `start`?

Ansible usually describes a **desired state**.

When you write:

```yaml
state: started
```

you are telling Ansible:

> Ensure that this service is running.

Ansible checks the current state:

- If the service is not running, Ansible starts it.
- If the service is already running, no change is required.

This behavior is related to **idempotency**.

---

# Idempotency

An operation is idempotent when repeating it produces the same desired final result.

For example:

```yaml
state: started
```

First execution:

```text
Service is stopped
        ↓
Ansible starts the service
```

Later execution:

```text
Service is already running
        ↓
No change is required
```

Most Ansible modules are designed around this principle.

Common service states include:

### `started`

```text
Not running → Start it
Already running → No change
```

### `stopped`

```text
Running → Stop it
Already stopped → No change
```

### `restarted`

```text
Restart the service and bring it back into a running state
```

Unlike `started`, `restarted` normally restarts a service even when it is already running.

---

# The `lineinfile` Module

The `lineinfile` module manages individual lines inside a file.

It can:

- Find a line
- Replace a matching line
- Add a line if it does not exist
- Ensure a line is absent

Example:

```yaml
- name: Ensure a DNS server is present in resolv.conf
  ansible.builtin.lineinfile:
    path: /etc/resolv.conf
    line: "nameserver 10.1.250.10"
    state: present
```

Ansible checks the current file contents:

- If the desired line is already present, no change is needed.
- If it is missing, Ansible adds it.

This makes `lineinfile` useful for managing specific configuration lines while preserving idempotency.

---

# Key Takeaways

- Ansible modules are reusable components that perform specific tasks on managed hosts.
- Modules are grouped by functionality and may be provided through Ansible collections.
- Examples include system, command, database, cloud, and Windows modules.
- `command` runs commands on managed hosts and supports free-form input.
- `chdir` changes the working directory before a command runs.
- `creates` can prevent a command from running when a specified path already exists.
- Prefer a dedicated module over a generic command when a suitable module exists.
- Some modules, such as `copy` and `service`, use named parameters.
- `script` transfers a script from the controller to managed hosts and executes it.
- `service` manages the desired state of a service.
- `started` means "ensure the service is running".
- Idempotency means repeated execution produces the same desired final state.
- `lineinfile` manages specific lines inside files.

## Further Reading

For the complete list of modules, module documentation, examples, and supported parameters, visit:

https://docs.ansible.com/ansible/latest/collections/index.html

