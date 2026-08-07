# Ansible YAML Basics

Notes from the KodeKloud course: *Ansible for the Absolute Beginner* (Section 2).

---

## What is YAML?

* Ansible playbooks are text files written in a format called **YAML**.
* YAML is commonly used to represent configuration data in a human-readable way.

Example:

```yaml
servers:
  - name: server1
    owner: john
    created: 2026-08-07
    status: active
```

---

## Key-Value Pairs

Basic YAML syntax:

```yaml
Fruit: Apple
Vegetable: Carrot
Liquid: Water
Meat: Chicken
```

### Rule

A key-value pair is written as:

```text
key: value
```

There must be a space **after** the colon.

---

## Lists (Arrays)

Use `-` for list items.

```yaml
Fruits:
  - Orange
  - Apple
  - Banana

Vegetables:
  - Carrot
  - Cauliflower
  - Tomato
```

Lists are used to store multiple items of the same type.

---

## Dictionaries (Maps)

A dictionary stores multiple properties for a single object.

```yaml
Banana:
  Calories: 105
  Fat: 0.4g
  Carbs: 27g

Grapes:
  Calories: 62
  Fat: 0.3g
  Carbs: 16g
```

Notice that the child properties are indented equally under each key.

---

## Indentation

Indentation is very important in YAML.

Correct:

```yaml
Banana:
  Calories: 105
  Fat: 0.4g
```

Incorrect:

```yaml
Banana:
 Calories: 105
    Fat: 0.4g
```

Use consistent indentation for related properties.

---

## List of Dictionaries

When each list item has its own properties, use a list of dictionaries.

```yaml
Fruits:
  - Name: Banana
    Calories: 105
    Fat: 0.4g
    Carbs: 27g

  - Name: Grapes
    Calories: 62
    Fat: 0.3g
    Carbs: 16g
```

This is a common structure in Ansible playbooks.

---

## Dictionary vs List vs List of Dictionaries

### Dictionary

Use when storing properties of a single object.

Example:

```yaml
Car:
  Brand: Toyota
  Model: Corolla
  Color: White
```

### List

Use when storing multiple items of the same type.

```yaml
Cars:
  - Sedan
  - SUV
  - Hatchback
```

### List of Dictionaries

Use when storing multiple objects, each with its own properties.

```yaml
Cars:
  - Brand: Toyota
    Model: Corolla

  - Brand: Honda
    Model: Civic
```

---

## Comments

Lines beginning with `#` are comments.

```yaml
# This is a comment
Fruit: Apple
```

Comments are ignored by Ansible and YAML parsers.

---

## Key Notes

* Ansible playbooks are written in YAML.
* YAML represents configuration data using indentation.
* Key-value pairs use the format `key: value`.
* Lists use the `-` symbol.
* Dictionaries store multiple properties for one object.
* Lists store multiple items.
* Lists of dictionaries store multiple objects with their own properties.
* Consistent indentation is essential.
* Comments begin with `#`.

---

## What I Learned

* YAML structure is determined by indentation.
* A small indentation mistake can change the meaning of the data.
* Ansible playbooks are easier to read when lists and dictionaries are formatted consistently.
* Understanding list vs dictionary vs list of dictionaries is important before writing real playbooks.

