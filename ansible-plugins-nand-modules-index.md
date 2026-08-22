# Ansible Plugins and the Modules & Plugins Index

## What Are Ansible Plugins?

Ansible plugins are pieces of code that extend or modify how Ansible works.

While **modules** generally perform actions on managed hosts, **plugins** extend Ansible's behaviour or influence how Ansible processes information and executes automation.

Plugins can extend different parts of Ansible, including:

- Inventory
- Action execution
- Callbacks
- Filters
- Lookups
- Connections
- Other internal Ansible functionality

> **Simple distinction:** Modules usually perform a task, while plugins extend or influence how Ansible works.

---

# Plugin Types

Different plugin types serve different purposes. Examples include:

- Inventory plugins
- Action plugins
- Callback plugins
- Lookup plugins
- Filter plugins
- Connection plugins
- Several other plugin types

When Ansible's built-in behaviour does not meet a particular requirement, the appropriate plugin type can provide an extension point.

---

# Inventory Plugins

## From Static Inventory to Dynamic Inventory

A static inventory might be maintained manually:

```ini
[webservers]
web1 ansible_host=192.0.2.10
web2 ansible_host=192.0.2.11
```

This is simple, but it can become outdated when infrastructure changes frequently.

For example, cloud instances may:

- Be created
- Be terminated
- Change addresses
- Change tags
- Move between logical groups

An inventory plugin can query an external source, such as a cloud provider, and dynamically build the inventory.

Conceptually:

```text
Ansible
   |
   v
Inventory Plugin
   |
   v
Cloud Provider API
   |
   v
Current Infrastructure Information
```

Depending on the plugin and its configuration, it may:

- Query the provider for current instances
- Filter instances, for example only running instances
- Build groups dynamically based on metadata such as tags

The main benefit is that the inventory can stay aligned with the current infrastructure without requiring every host to be maintained manually.

---

# Custom Modules

Ansible can be extended with custom modules when existing modules do not provide the required functionality.

For example, a custom module could interact with a cloud provider API to provision resources using organization-specific requirements, such as:

- Specific image or AMI versions
- Instance types
- Security group settings
- Other required configuration

Conceptually:

```text
Playbook Task
     |
     v
Custom Module
     |
     v
Cloud Provider API
     |
     v
Provisioned Resource
```

> **Important correction:** A custom module and a plugin are different extension mechanisms. Modules perform tasks, while plugins extend or modify Ansible's behaviour. A custom module should not generally be called a "module plugin."

---

# Action Plugins

Action plugins control or extend how certain tasks are executed.

They can perform processing on the controller before, during, or around module execution.

For example, a custom action plugin could coordinate a high-level task that involves:

- Load balancer configuration
- API calls
- SSL certificate operations
- Health checks

Conceptually:

```text
High-Level Playbook Task
          |
          v
Action Plugin
          |
          +-- API Operations
          +-- Additional Processing
          +-- Module Execution
```

This can hide underlying complexity and provide consistent behaviour for repeated automation tasks.

---

# Lookup Plugins

Lookup plugins retrieve data from sources outside the current playbook.

Possible sources include:

- Files
- Databases
- APIs
- Secret-management systems
- Key-value stores

They allow external data to be retrieved and used during Ansible processing.

Conceptually:

```text
External Data Source
        |
        v
Lookup Plugin
        |
        v
Data Used by Ansible
```

Lookup plugins are commonly used with Jinja expressions and Ansible's `lookup()` or `query()` functions.

---

# Filter Plugins

Filter plugins transform or format data.

Conceptually:

```text
Input Data
    |
    v
Filter Plugin
    |
    v
Transformed Data
```

They are useful when built-in Jinja or Ansible filters do not provide the required transformation.

---

# Connection Plugins

Connection plugins determine how Ansible communicates with managed targets.

Examples include:

- SSH
- WinRM
- Local execution
- Other supported connection mechanisms

Conceptually:

```text
Ansible Controller
       |
       v
Connection Plugin
       |
       v
Managed Target
```

For example, Linux hosts commonly use SSH, while Windows hosts can use WinRM.

---

# Callback Plugins

Callback plugins hook into events that occur during an Ansible run.

They can be used for:

- Additional logging
- Notifications
- Reporting
- Capturing execution events
- Customizing output

Conceptually:

```text
Ansible Run
    |
    +-- Task Started
    +-- Task Completed
    +-- Task Failed
    |
    v
Callback Plugin
    |
    v
Custom Logging / Reporting / Notifications
```

---

# When to Use Which Plugin Type

A simple way to think about plugin selection:

- Need to retrieve external data -> **Lookup plugin**
- Need to transform data -> **Filter plugin**
- Need to communicate with a target differently -> **Connection plugin**
- Need custom execution output or reporting -> **Callback plugin**
- Need dynamic infrastructure discovery -> **Inventory plugin**
- Need to extend how a task is executed -> **Action plugin**

---

# Ansible Modules and Plugins Index

The Ansible Modules and Plugins Index helps you find available Ansible content and documentation.

It is useful when you know what you want to automate but do not know the exact module or plugin required.

The index can help with:

- Searching by keyword
- Browsing by category
- Finding collections
- Viewing documentation
- Checking options and examples

---

# Collections Come First

Ansible content is commonly distributed through **collections**.

Instead of looking only for an individual module, first identify the collection that provides the required functionality.

For example:

```text
cisco.ios
```

Modules in that collection include names such as:

```text
cisco.ios.ios_config
cisco.ios.ios_facts
```

A Fully Qualified Collection Name (FQCN) generally follows this format:

```text
namespace.collection.plugin_or_module
```

For example:

```text
cisco.ios.ios_config
```

Breaking it down:

```text
cisco       -> Namespace
ios         -> Collection
ios_config  -> Module
```

Using the FQCN in a playbook makes it clear exactly which collection provides the module.

Example:

```yaml
- name: Configure a Cisco IOS device
  cisco.ios.ios_config:
    lines:
      - hostname Router1
```

---

# Installing a Collection

Collections can be installed using `ansible-galaxy`.

Example:

```bash
ansible-galaxy collection install cisco.ios
```

After installation, the content from that collection can be used subject to its requirements and the Ansible environment configuration.

---

# Using `ansible-doc`

The `ansible-doc` command can display documentation for installed Ansible content.

Example:

```bash
ansible-doc cisco.ios.ios_config
```

It can provide information such as:

- Description
- Parameters and options
- Usage information
- Examples

To list available modules:

```bash
ansible-doc -l
```

To filter results for a particular collection:

```bash
ansible-doc -l cisco.ios
```

> The exact output depends on which collections and plugins are installed in the current Ansible environment.

---

# Version Compatibility

Version compatibility matters when using Ansible collections.

Before upgrading:

- Review compatibility requirements
- Read release notes
- Test in a non-production environment
- Check compatibility between `ansible-core` and the required collections

For infrastructure-specific collections, it is safer to test compatible versions together rather than upgrading components independently without validation.

---

# Why the Index Matters

## 1. Search and Filtering

Search for content relevant to a particular task instead of guessing module or plugin names.

## 2. Detailed Documentation

Documentation can explain:

- Purpose
- Parameters
- Return values
- Requirements
- Examples

## 3. Version and Compatibility Information

Collection and documentation information can help identify requirements and compatibility considerations.

## 4. Community and Collection Content

Ansible's ecosystem includes content maintained by communities, vendors, and organizations through collections.

This makes collections an important way to discover functionality beyond `ansible.builtin`.

---

# Key Takeaways

- Plugins extend or modify how Ansible behaves.
- Modules generally perform actions, while plugins influence Ansible's processing or behaviour.
- Inventory plugins can dynamically discover infrastructure.
- Action plugins can extend task execution behaviour.
- Lookup plugins retrieve external data.
- Filter plugins transform data.
- Connection plugins control communication with targets.
- Callback plugins can customize logging, notifications, reporting, and output.
- Ansible content is commonly distributed through collections.
- An FQCN generally follows `namespace.collection.plugin_or_module`.
- `ansible-galaxy collection install` installs collections.
- `ansible-doc` helps view documentation for installed Ansible content.
- The Modules and Plugins Index helps discover the correct content, documentation, and collections.

---

## Further Reading

Official Ansible documentation:

- https://docs.ansible.com/ansible/latest/plugins/plugins.html
- https://docs.ansible.com/ansible/latest/collections/index.html
- https://docs.ansible.com/ansible/latest/cli/ansible-doc.html
- https://docs.ansible.com/ansible/latest/collections_guide/collections_using.html

