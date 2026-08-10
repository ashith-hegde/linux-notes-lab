# Ansible Inventory Notes

## What is an inventory?

* Ansible can work with **one or multiple systems** in an infrastructure at the same time.
* To manage remote systems, Ansible must establish connectivity to those systems.
* Common connection methods:

  * **Linux:** SSH
  * **Windows:** PowerShell Remoting (WinRM)

This is what makes Ansible **agentless**.

### Agentless

Agentless means that **no additional Ansible agent needs to be installed on the target machines**.

A common disadvantage of many other orchestration tools is that an agent must be installed and configured on every managed system before automation can be executed.

---

## Inventory file

Information about target systems is stored in an **inventory file**.

If a custom inventory is not provided, Ansible uses the default inventory:

```text
/etc/ansible/hosts
```

The inventory format is similar to an INI file.

### Simple inventory

```ini
server1.company.com
server2.company.com
server3.company.com
```

---

## Grouping hosts

Hosts can be grouped together.

```ini
[mail]
server1.company.com
server3.company.com
```

* Group names are written inside **square brackets**.
* Hosts belonging to that group are listed below the group name.
* Multiple groups can exist in the same inventory file.

Example:

```ini
[web]
web1.company.com
web2.company.com

[db]
db1.company.com
```

---

## Host aliases

Aliases can be assigned to hosts.

```ini
web  ansible_host=server1.company.com
db   ansible_host=server2.company.com
mail ansible_host=server3.company.com
web2 ansible_host=server4.company.com
```

* `web`, `db`, and `mail` are aliases.
* `ansible_host` specifies the real hostname or IP address.

---

# Important inventory variables

| Variable             | Purpose                     | Example                           |
| -------------------- | --------------------------- | --------------------------------- |
| `ansible_connection` | Connection type             | `ssh`, `winrm`, `local`           |
| `ansible_port`       | Port number                 | `22`, `5985`, `5986`              |
| `ansible_user`       | Remote username             | `root`, `ubuntu`, `administrator` |
| `ansible_ssh_pass`   | SSH password (Linux)        | `mypassword`                      |
| `ansible_password`   | Generic connection password | `mypassword`                      |

---

## `ansible_connection`

Defines how Ansible connects to the target system.

Examples:

```ini
localhost ansible_connection=local
win1 ansible_connection=winrm
```

---

## `ansible_port`

Defines the port used for the connection.

Common defaults:

* **SSH:** `22`
* **WinRM HTTP:** `5985`
* **WinRM HTTPS:** `5986`

---

## `ansible_user`

Defines the user used for remote connections.

Example:

```ini
web ansible_user=ubuntu
```

For Linux systems, the user is often `root` or another administrative account.

---

## `ansible_ssh_pass`

Defines the SSH password for Linux hosts.

Example:

```ini
web ansible_ssh_pass=secret
```
---

## `ansible_password`

Defines a generic connection password used by some connection plugins, especially **WinRM for Windows hosts**.

Example:

```ini
win1 ansible_connection=winrm ansible_user=administrator ansible_password=secret
```

This variable is commonly used with Windows systems managed through WinRM. As with SSH passwords, storing plaintext passwords in inventory files is not recommended.
### Security note

Storing passwords directly in inventory files is **not recommended**.

### Best practice

* Use **SSH key-based passwordless authentication** between the control node and managed hosts.
* This is the preferred approach in production and corporate environments.

For sensitive data such as passwords, Ansible provides **Ansible Vault** to encrypt secrets.

---

# Key Takeaways

* Inventory tells Ansible **which hosts to manage**.
* Hosts can be organized into **groups**.
* Aliases make inventories easier to read.
* Connection behavior is controlled through inventory variables.
* Production environments should use **SSH keys and Ansible Vault instead of plaintext passwords**.

