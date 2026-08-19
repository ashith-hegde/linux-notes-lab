# Ansible Playbooks

## What is an Ansible Playbook?

Ansible Playbooks are Ansible's **orchestration language**.

A playbook defines **what Ansible should do, which hosts it should perform the work on, and in what sequence the tasks should be executed**.

A playbook is written in **YAML**.

A simple way to think about it:

```text
Playbook
   |
   +-- Play 1
   |     |
   |     +-- Task 1
   |     +-- Task 2
   |
   +-- Play 2
         |
         +-- Task 1
         +-- Task 2
```

---

## Playbook Structure

An Ansible playbook is normally a single YAML file containing one or more **plays**.

```yaml
---
- name: Play 1
  hosts: localhost
  tasks:

    - name: Execute command 'date'
      ansible.builtin.command: date

    - name: Execute script on server
      ansible.builtin.script: test_script.sh

- name: Play 2
  hosts: webservers
  tasks:

    - name: Install web service
      ansible.builtin.yum:
        name: httpd
        state: present

    - name: Start web server
      ansible.builtin.service:
        name: httpd
        state: started
```

> The `ansible.builtin.*` form is the fully qualified collection name (FQCN). It makes the module being used explicit.

---

## Playbook

A **playbook** is the complete YAML file containing one or more plays.

Example:

```text
site.yml
```

The playbook describes the overall automation workflow.

---

## Play

A **play** maps a set of tasks to a particular group of hosts.

```yaml
- name: Configure web servers
  hosts: webservers
  tasks:
    ...
```

A play can be thought of as:

> "Perform these tasks against these hosts."

A playbook can contain multiple plays targeting different hosts.

---

## Task

A **task** is an individual unit of work inside a play.

```yaml
- name: Install Apache
  ansible.builtin.yum:
    name: httpd
    state: present
```

Each task normally invokes a module to perform the requested action.

---

## Module

A **module** performs the actual operation requested by a task.

Examples:

| Module | Purpose |
|---|---|
| `command` | Execute a command |
| `shell` | Execute a command through a shell |
| `script` | Transfer and execute a local script |
| `yum` | Manage packages on YUM-based systems |
| `dnf` | Manage packages using DNF |
| `service` | Manage services |
| `copy` | Copy files to managed hosts |
| `file` | Manage files, directories and permissions |
| `user` | Manage users |
| `debug` | Display information during execution |

Ansible provides hundreds of modules. The official Ansible module documentation should be used to check available parameters and behavior.

---

## Understanding the Relationship

```text
Playbook
    |
    +-- Play
          |
          +-- hosts
          |
          +-- tasks
                |
                +-- Task
                      |
                      +-- Module
```

For example:

```yaml
- name: Install Apache
  hosts: webservers

  tasks:
    - name: Install Apache package
      ansible.builtin.yum:
        name: httpd
        state: present
```

Here:

- **Playbook** → the YAML file
- **Play** → `Install Apache`
- **Hosts** → `webservers`
- **Task** → `Install Apache package`
- **Module** → `ansible.builtin.yum`
- **Module parameters** → `name` and `state`

---

## The `hosts` Parameter

The `hosts` parameter specifies which inventory hosts the play should run against.

```yaml
- name: Configure web servers
  hosts: webservers
  tasks:
    ...
```

If the inventory contains:

```ini
[webservers]
web1
web2
web3
```

then the play targets all three hosts.

```text
Play
 |
 +-- web1
 +-- web2
 +-- web3
```

The target hosts are selected at the **play level**.

---

## Multiple Plays in One Playbook

```yaml
---
- name: Configure web servers
  hosts: webservers

  tasks:
    - name: Install Apache
      ansible.builtin.yum:
        name: httpd
        state: present

- name: Configure database servers
  hosts: databases

  tasks:
    - name: Install PostgreSQL
      ansible.builtin.yum:
        name: postgresql-server
        state: present
```

This allows one playbook to orchestrate changes across different parts of an infrastructure.

---

## Common Operations Performed by Playbooks

Playbooks can automate operations such as:

- Execute commands
- Run scripts
- Install or remove packages
- Start, stop or restart services
- Create users
- Copy configuration files
- Modify files
- Create directories
- Manage permissions
- Configure applications
- Deploy software
- Restart systems when required

Example:

```yaml
- name: Configure application server
  hosts: appservers

  tasks:

    - name: Install required package
      ansible.builtin.yum:
        name: httpd
        state: present

    - name: Start web service
      ansible.builtin.service:
        name: httpd
        state: started

    - name: Create application directory
      ansible.builtin.file:
        path: /opt/myapp
        state: directory
        mode: '0755'
```

---

## Running a Playbook

Basic syntax:

```bash
ansible-playbook <playbook-file>
```

Example:

```bash
ansible-playbook playbook.yml
```

With a custom inventory:

```bash
ansible-playbook -i inventory.ini playbook.yml
```

---

## Getting Help

```bash
ansible-playbook --help
```

This shows additional command-line options without requiring them to be memorized.

---

## Checking Playbook Syntax

Before running a playbook:

```bash
ansible-playbook --syntax-check playbook.yml
```

This is useful for catching YAML/Ansible syntax problems before execution.

---

## Listing Hosts

```bash
ansible-playbook -i inventory.ini playbook.yml --list-hosts
```

This is useful when working with groups and larger inventories.

---

## Listing Tasks

```bash
ansible-playbook playbook.yml --list-tasks
```

This shows the tasks contained in the playbook without executing them.

---

## Ad-hoc Commands vs Playbooks

### Ad-hoc command

```bash
ansible localhost -m ansible.builtin.command -a "date"
```

Ad-hoc commands are useful for:

- Quick tests
- One-time operations
- Checking connectivity
- Simple troubleshooting

### Playbook

```bash
ansible-playbook -i inventory.ini playbook.yml
```

Playbooks are better for:

- Multiple tasks
- Repeatable automation
- Structured configuration
- Variables
- Conditions
- Loops
- Handlers
- Multiple plays
- Version-controlled automation

A useful mental model:

```text
Ad-hoc command
    ↓
Quick / one-off operation

Playbook
    ↓
Repeatable / structured automation
```

---

## YAML Structure and Indentation

Because playbooks are YAML files, **indentation is significant**.

```yaml
- name: Configure web server
  hosts: webservers

  tasks:
    - name: Install Apache
      ansible.builtin.yum:
        name: httpd
        state: present
```

The hierarchy is:

```text
Play
 ├── name
 ├── hosts
 └── tasks
      └── task
           ├── name
           └── module
                ├── name
                └── state
```

Incorrect indentation can change the YAML structure or cause the playbook to fail.

Use consistent spaces rather than tabs.

---

## Key Takeaways

- An **Ansible playbook** is a YAML file containing one or more plays.
- A **play** defines which hosts should receive a set of tasks.
- A **task** is an individual unit of work.
- A **module** performs the actual operation for a task.
- The `hosts` parameter is defined at the play level.
- A playbook can contain multiple plays targeting different host groups.
- Playbooks make automation repeatable, structured and version-controllable.
- `ansible-playbook` executes playbooks.
- `--syntax-check` checks syntax before execution.
- `--list-hosts` shows targeted hosts.
- `--list-tasks` shows tasks without executing them.
- YAML indentation determines the structure of the playbook.

## Basic Mental Model

```text
Inventory
   ↓
Which machines?

Playbook
   ↓
What should happen?

Play
   ↓
Which hosts receive these tasks?

Task
   ↓
What individual action should happen?

Module
   ↓
How is that action performed?
```

# Ansible Playbooks — Verification

## Why Verify Playbooks?

Running an untested playbook directly against production systems can introduce unexpected changes.

Possible consequences include:

- Service downtime
- Incorrect configuration
- Data loss
- Application failures
- Security issues
- Difficult rollback or recovery

Verification is similar to a **rehearsal before the real operation**.

The goal is to identify syntax errors, unexpected changes, and potential problems before making changes to production systems.

A useful workflow is:

```text
Write playbook
      ↓
Syntax check
      ↓
Lint
      ↓
Check mode
      ↓
Diff mode (when appropriate)
      ↓
Test in a safe environment
      ↓
Production
```

---

# Syntax Check

Syntax checking verifies that the playbook can be parsed correctly and that there are no basic YAML/Ansible syntax problems.

Use:

```bash
ansible-playbook --syntax-check playbook.yml
```

Example:

```bash
ansible-playbook --syntax-check site.yml
```

Syntax checking does **not** prove that the playbook will work correctly. For example, the YAML may be valid while a variable is missing, a module parameter is incorrect, a service does not exist, a path is wrong, or a remote host is unreachable.

Therefore, syntax checking is only the first verification step.

---

# Check Mode

Check mode is Ansible's **dry-run mode**.

It allows Ansible to evaluate tasks without applying supported changes to the target systems.

```bash
ansible-playbook --check playbook.yml
```

With an inventory:

```bash
ansible-playbook -i inventory.ini playbook.yml --check
```

Example:

```yaml
- name: Install Apache
  hosts: webservers

  tasks:
    - name: Ensure Apache is installed
      ansible.builtin.yum:
        name: httpd
        state: present
```

Running:

```bash
ansible-playbook playbook.yml --check
```

allows you to inspect what Ansible believes would change without normally making the actual change.

### Important limitation

**Not every Ansible module fully supports check mode.**

If a module does not support check mode, its behavior may differ from a normal execution and some tasks may be skipped.

Therefore, check mode should not be treated as a guarantee that production execution will behave exactly the same.

---

# Diff Mode

Diff mode displays differences between the current state and the state Ansible intends to create for modules that support diff output.

```bash
ansible-playbook --diff playbook.yml
```

It is particularly useful when modifying files or configuration.

Using it with check mode is often useful:

```bash
ansible-playbook --check --diff playbook.yml
```

```text
--check
   ↓
Don't normally apply changes

--diff
   ↓
Show relevant before/after differences

Together
   ↓
Review the expected impact before execution
```

### Example use case

Suppose Ansible manages:

```text
/etc/httpd/conf/httpd.conf
```

A check + diff run may show the configuration lines that would change.

This gives the administrator an opportunity to review the change before applying it.

### Security consideration

Be careful when using `--diff` with sensitive files because differences can potentially expose secrets or sensitive configuration values in terminal output or logs.

---

# Ansible Lint

`ansible-lint` is a command-line tool used to analyze Ansible content such as:

- Playbooks
- Roles
- Collections

It checks for potential problems including:

- Common errors
- Suspicious constructs
- Deprecated patterns
- Style issues
- Best-practice violations

Basic syntax:

```bash
ansible-lint playbook.yml
```

Example:

```bash
ansible-lint site.yml
```

A linter does not execute the playbook. It analyzes the code and reports issues.

---

# What is Linting?

**Linting** is the process of using a specialized tool called a **linter** to analyze source code for potential:

- Errors
- Bugs
- Style violations
- Suspicious constructs
- Best-practice violations

Linting helps identify problems early, before the code is executed.

---

# Verification Commands — Quick Reference

| Purpose | Command |
|---|---|
| Syntax check | `ansible-playbook --syntax-check playbook.yml` |
| Check mode | `ansible-playbook --check playbook.yml` |
| Diff mode | `ansible-playbook --diff playbook.yml` |
| Check + diff | `ansible-playbook --check --diff playbook.yml` |
| Lint | `ansible-lint playbook.yml` |
| Execute | `ansible-playbook playbook.yml` |

---

# Practical Verification Workflow

For a playbook that will eventually be used in production:

## 1. Check syntax

```bash
ansible-playbook --syntax-check site.yml
```

## 2. Run ansible-lint

```bash
ansible-lint site.yml
```

## 3. Run check mode

```bash
ansible-playbook -i inventory.ini site.yml --check
```

## 4. Add diff mode when configuration/file changes are involved

```bash
ansible-playbook -i inventory.ini site.yml --check --diff
```

## 5. Test against a controlled environment

Use a lab, development environment, or other non-production target where possible.

## 6. Review the results

Confirm:

- Correct hosts are targeted
- Variables have the expected values
- Tasks behave as expected
- No unexpected configuration changes are reported
- Sensitive information is not exposed through output

## 7. Execute in production

Only after the playbook has been appropriately tested and reviewed:

```bash
ansible-playbook -i inventory.ini site.yml
```

---

# Important Limitation of Verification

No single verification option guarantees that a playbook is completely safe.

```text
Syntax check
    ↓
Checks syntax

Lint
    ↓
Checks code quality and common problems

Check mode
    ↓
Predicts changes where supported

Diff mode
    ↓
Shows relevant differences where supported

Testing
    ↓
Validates actual behavior in a controlled environment

Production execution
    ↓
Real change
```

Each step catches a different class of problem.

---

# Key Takeaways

- Verify playbooks before running them against production systems.
- `--syntax-check` checks basic playbook syntax.
- `--check` performs a dry run for modules that support check mode.
- `--diff` displays relevant before/after differences.
- `--check --diff` is useful for reviewing configuration changes before applying them.
- `ansible-lint` checks playbooks for errors, suspicious constructs and best-practice issues.
- Check mode is **not supported equally by every module**.
- Verification does not replace testing in a controlled environment.
- Be careful with `--diff` because sensitive information can potentially appear in output.
- A production-ready workflow should combine syntax checking, linting, dry runs, review and controlled testing.

## Verification Mental Model

```text
Syntax Check
     ↓
"Can Ansible parse this?"

Lint
     ↓
"Does this follow good Ansible practices?"

Check Mode
     ↓
"What changes are expected?"

Diff Mode
     ↓
"What exactly will change?"

Controlled Testing
     ↓
"Does it actually behave correctly?"

Production
     ↓
"Apply the verified automation"
```


