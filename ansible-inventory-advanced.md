# Ansible Advanced Inventory Notes

## Why do we need different inventory formats?

Different environments have different inventory requirements.

### Small startup

* Few servers
* Simple roles such as:

  * Web hosting
  * Database server
* A simple INI inventory is usually sufficient.

### Large enterprise / multinational company

* Hundreds of servers
* Multiple data centers
* Different regions (US, Europe, Asia)
* Multiple application teams
* More detailed structure is required.

In larger environments, a hierarchical inventory is easier to maintain and scale.

---

# Supported inventory formats

Ansible supports two common inventory formats:

* **INI**
* **YAML**

---

# INI inventory

INI is simple and easy to read.

```ini
[webservers]
web1.example.com
web2.example.com

[dbservers]
db1.example.com
```

This format works well for small and medium environments.

---

# YAML inventory

YAML is more structured and flexible.

```yaml
all:
  children:
    webservers:
      hosts:
        web1.example.com:
        web2.example.com:

    dbservers:
      hosts:
        db1.example.com:
```

YAML becomes especially useful when inventories grow larger.

---

# When should I choose YAML?

Choose YAML when you need:

* nested groups,
* host variables,
* group variables,
* location-based organization,
* environment-based organization (dev/test/prod),
* clearer hierarchical structure.

---

# Grouping hosts

Grouping allows Ansible to manage multiple hosts together.

Example:

```ini
[webservers]
web1.example.com
web2.example.com
```

Running:

```bash
ansible webservers -i inventory.ini -m ping
```

targets both web servers at once.

---

# Parent-child relationships

Suppose web servers exist in multiple regions:

* US
* Europe

Many settings are common to all web servers, while some are region-specific.

Instead of duplicating configuration, create:

* a **parent group**: `webservers`
* child groups:

  * `webservers_us`
  * `webservers_eu`

---

# Parent-child in INI format

```ini
[webservers:children]
webservers_us
webservers_eu

[webservers_us]
us-web1.example.com
us-web2.example.com

[webservers_eu]
eu-web1.example.com
eu-web2.example.com
```

The `:children` suffix defines child groups.

---

# Parent-child in YAML format

```yaml
all:
  children:
    webservers:
      children:
        webservers_us:
          hosts:
            us-web1.example.com:
              ansible_host: 192.168.8.101
            us-web2.example.com:
              ansible_host: 192.168.8.102

        webservers_eu:
          hosts:
            eu-web1.example.com:
              ansible_host: 10.10.1.101
            eu-web2.example.com:
              ansible_host: 10.10.1.102
```

* `hosts` defines hosts inside a group.
* `children` defines child groups.

---

# Real-world benefit

Parent-child inventories help:

* avoid configuration duplication,
* apply common settings once,
* keep inventories organized,
* scale to large environments more easily.

Example:

* Common Nginx configuration → parent group
* US timezone → `webservers_us`
* EU timezone → `webservers_eu`

---

# Key Takeaways

* INI is simple; YAML is structured.
* YAML is preferred for large or complex inventories.
* Groups allow bulk operations.
* Parent-child relationships reduce duplication.
* Hierarchical inventories improve maintainability in enterprise environments.

