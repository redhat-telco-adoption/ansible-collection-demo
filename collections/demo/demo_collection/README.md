# Ansible Collection - demo.demo_collection

This collection contains a demo role to showcase how to create and use custom Ansible collections within Ansible Automation Platform.

## Contents

- [Ansible Collection - demo.demo_collection](#ansible-collection---demodemo_collection)
  - [Contents](#contents)
  - [Installation](#installation)
  - [Usage](#usage)
  - [Roles](#roles)
  - [Playbooks](#playbooks)

## Installation

To install this collection, use the following command:

```
ansible-galaxy collection install demo.demo_collection
```

Note: In a production environment, this collection would typically be installed from a Private Automation Hub.

## Usage

To use the role from this collection in your playbooks, you can specify the fully-qualified collection name:

```yaml
- hosts: all
  tasks:
    - name: Include demo_role
      include_role:
        name: demo.demo_collection.demo_role
```

Alternatively, you can add the collection to the play's `collections` element:

```yaml
- hosts: all
  collections:
    - demo.demo_collection
  tasks:
    - name: Include demo_role
      include_role:
        name: demo_role
```

## Roles

This collection contains the following roles:

- `demo_role`: A simple role that sets and displays a demo message.

## Playbooks

The `playbook/main.yml` file in this collection demonstrates how to use the `demo_role`:

```yaml
---
- name: Demo playbook using our collection
  hosts: localhost
  collections:
    - demo.demo_collection
  tasks:
    - name: Use the demo role
      include_role:
        name: demo_role
    
    - name: Display the demo message
      debug:
        msg: "{{ demo_message }}"
```