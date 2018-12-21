ansible-development-kit
=========

## How to use:

1. pip install cookiecutter

### Create a new role

1. cookiecutter https://github.com/lean-delivery/ansible-development-kit
_Alternative way:_
1. molecule init template --url https://github.com/lean-delivery/ansible-development-kit

### Update an existing role

1. cd ansible-role-example
1. cookiecutter https://github.com/lean-delivery/ansible-development-kit --output-dir .. --overwrite-if-exists
1. git status
1. git add . -p
1. git commit -m "Updated by cookiecutter and ansible-development-kit"
