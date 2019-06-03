---
layout: default
title: Ansible Code Snippets
nav_order: 3
has_children: false
permalink: /docs/ansible_code_snippets
---

# Ansible Code Snippets
{: .no_toc }

This document demonstrates useful code snippets for Ansible roles/playbooks. These snippets are provided for Ansible role authors and contributors to avoid repetitive typing for typical tasks and steps. 

## Table of contents
{: .no_toc .text-delta }

*  TOC
{:toc}

---

## Tasks and Variables Separation

When working with multiple OS versions/distributions (RedHat vs Debian for instance), there are packages, configuration files and parameters that are vastly different than one another. There can even be instances in which sub-versions of OSes can have major differences, such as service vs. systemctl in CentOS 6 vs. 7 etc.

There are different ways to handle these issues both in the same playbook or play or by separating out tasks to other task files.

### Variables Separation

```yaml {% raw %}
# bad
- name: Set path
  set_fact:
    my_var: /tmp/mypath
  when: ansible_distribution == 'CentOS'
 
# good
- name: Load a variable file based on the OS type, or a default if not found
  include_vars: '{{ platform_vars }}'
  with_first_found:
    - '{{ ansible_os_family }}.yml'
    - '{{ ansible_distribution }}.yml'
    - default.yml
  loop_control:
    loop_var: platform_vars

- name: Load a variable file based on the service manager
  include_vars: '{{ service_manager }}'
  with_first_found:
    - '{{ ansible_service_mgr }}.yml'
    - systemv.yml
  loop_control:
    loop_var: service_manager
```

### Tasks Separation

```yaml {% raw %}
# bad
- name: ensure that directory for apache is ready
  file:
    path: '{{ apache_dir }}'
    owner: '{{ web_server_user }}'
    group: '{{ web_server_group }}'
    recurse: true
    state: directory
  become: true
  when: ansible_os_family == 'RedHat'
 
# good
- name: Configure and install packages for current OS
  include_tasks: '{{ platform_tasks }}'
  with_first_found:
    - '{{ ansible_os_family }}.yml'
    - not_supported.yml
  loop_control:
    loop_var: platform_tasks
```

### Why?
{: .no_toc }

The way of tasks and variables separation using dynamic includes is more flexible and it's easier to support. For example, to add new OS support you just need to put an additional file into Ansible role with corresponding name and content without changing the exact code of Ansible role or playbook. Thus avoid separation of code and variables by using `when` conditionals in playbooks.

---

## Install requirements

```yaml {% raw %}
- name: Install requirements
  package:
    name: '{{ requirements }}'
    state: present
  register: installed_packages
  until: installed_packages is succeeded
  become: true
 
- name: Install required packages
  package:
    name: '{{ packages_base | union(packages_additional) | unique }}'
    state: present
  register: installed_packages
  until: installed_packages is succeeded
  become: true
```

---

## Selinux support

```yaml {% raw %}
---
- name: install ansible selinux support library
  become: true
  package:
    name: libselinux-python
    state: present
  register: installed_package
  until: installed_package is succeeded
 
- name: Install ansible selinux configure libraries
  become: true
  package:
    name:
      - policycoreutils-python
      - libsemanage-python
    state: present
  register: installed_package
  until: installed_package is succeeded
  when: ansible_selinux.status == "enabled"
 
- name: Enable connections to HTTP port
  become: true
  seport:
    ports: "{{ selinux_ports }}"
    proto: tcp
    setype: http_port_t
    state: present
  when:
    - ansible_selinux.status == "enabled"
    - ansible_selinux.mode != "disabled" 
```

---

## Conditionals

### Avoid comparison to empty string and 'true/false'

Use `when: var` rather than `when: var == True` (or conversely `when: not var`)
Use `when: var` rather than `when: var != ""` (or conversely `when: not var` rather than `when: var == ""`)


### Why?
{: .no_toc }

Ansible came from Python world. It's supposed to follow `Zen of Python` principles and related codestyle conventions. As for conditionals, Python code guides recommend to use direct truth value testing.

