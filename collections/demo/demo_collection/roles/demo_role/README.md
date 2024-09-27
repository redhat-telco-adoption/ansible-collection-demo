# Ansible Role: demo_role

This role is part of the `demo.demo_collection` and serves as a simple demonstration of role functionality within a custom Ansible collection.

## Role Variables

This role sets the following variable:

- `demo_message`: A string containing a greeting message.

## Example Playbook

```yaml
- hosts: all
  collections:
    - demo.demo_collection
  roles:
    - demo_role
  tasks:
    - name: Display the demo message
      debug:
        msg: "{{ demo_message }}"
```

## Role Tasks

The role performs the following tasks:

1. Sets a fact `demo_message` with a greeting string.
2. Displays the `demo_message` using the `debug` module.

## Requirements

This role is intended to be used as part of the `demo.demo_collection`. It has no external dependencies.