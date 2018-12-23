ansible-development-kit
=========

## How to use:

pip install cookiecutter

### Create a new role

- cookiecutter https://github.com/lean-delivery/ansible-development-kit

or

- molecule init template --url https://github.com/lean-delivery/ansible-development-kit

Enter for the role name question a value without the ansible-role- prefix, e.g. example.

To be able to update the role with cookiecutter and run tests locally, rename created directory adding the ansible-role- prefix.

- mv example ansible-role-example

Make changes in the corresponding files: copyright section in LICENSE, badge section in README.md
(you can get galaxy's role id by running: `ansible-galaxy info lean_delivery.example |grep '\bid'`), etc.

### Update an existing role

1. cd ansible-role-example
1. cookiecutter https://github.com/lean-delivery/ansible-development-kit --output-dir .. --overwrite-if-exists
1. git status
1. git add . -p
1. git commit -m "Updated by cookiecutter and ansible-development-kit"

In order not to provide the same answers for cookecutter's questions it makes sense to put in the role's directory a config file `.cookiecutter.yml` like this:

```yaml
---
default_context:
  role_name: ansible-role-example
```

and run cookiecutter the following way:

cookiecutter https://github.com/lean-delivery/ansible-development-kit --output-dir .. --overwrite-if-exists --config-file .cookiecutter.yml --no-input
