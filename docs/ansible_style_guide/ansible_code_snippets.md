---
layout: default
title: Ansible Code Snippets
nav_order: 3
has_children: false
permalink: /docs/ansible_style_guide
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

```yaml
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

```yaml
# bad
- name: ensure that directory for apache is ready
  file:
    path: '{{ apache_dir }}'
    owner: '{{ web_server_user }}'
    group: '{{ web_server_group }}'
    recurse: True
    state: directory
  become: True
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

The way of tasks and variables separation using dynamic includes is more flexible and it's easier to support. For example, to add new OS support you just need to put an additional file into Ansible role with corresponding name and content without changing the exact code of Ansible role or playbook.

### Why Doesn't Your Style Follow Theirs?

The script examples are inconsistent in style throughout the Ansible documentation; the purpose of this document is to define a consistent style that can be used throughout Ansible scripts to create robust, readable code.

---

## Start of Files

All YAML files (regardless of their association with Ansible or not) should begin with `---` to define the document start. 

### Why?
{: .no_toc }

It's better processed by linters and parsers. It allows easily indicate start of new yaml objects and documents.

---

## End of Files

You should always end your files with a newline.

### Why?
{: .no_toc }

This is common Unix best practice, and avoids any prompt misalignment when printing files in a terminal.

---

## Quotes

**You should only quote strings when it is absolutely necessary** or required by YAML, and then use single quotes. 
Single quotes are preferrable to double since they are shorter and require less efforts to support, especially for roles supporting Windows.

### Double quotes should be used in the following cases:
{: .no_toc }

1.  When they are nested within single quotes (e.g. Jinja map reference)

    ```yaml {% raw %}
    - name: start all services
      service:
        name: '{{ item["service_name"] }}'
        state: started
        enabled: true
      loop: '{{ services }}' {% endraw %}
    ```

2.  When your string requires escaping characters (e.g. using "\n" to represent a newline)

    ```yaml
    # double quotes to escape characters
    - name print text with two lines
      debug:
        msg: "Line one\nLine two"
    ```

3.  When you need to split long string without spaces to several lines:

    ```yaml
    # long string without spaces
    - name Set plugin url 
      set_fact:
        plugin_jar: "https://github.com/checkstyle/\
                      sonar-checkstyle/releases/download/4.17/\
                      checkstyle-sonar-plugin-4.17.jar"
    ```

### Why?
{: .no_toc }

Even though strings are the default type for YAML, syntax highlighting looks better when explicitly set types. This also helps troubleshoot malformed strings when they should be properly escaped to have the desired effect.

---

## Long strings

If you write a long string containing whitespaces, it's preferrable to use the "Literal Block Scalar" style and omit all special quoting. Values can span multiple lines using `|` or `>`. Spanning multiple lines using a "Literal Block Scalar" (`|`) will include the newlines and any trailing spaces. Using a "Folded Block Scalar" (`>`) will fold newlines to spaces; it’s used to make what would otherwise be a very long line easier to read and edit:

```yaml {% raw %}
java_folder: >-
  {{ (java_major_version|int <= 8)
    | ternary(java_package + '1.' + java_major_version|string + '.0_' + java_minor_version|string,
              java_package + '-'  + java_major_version|string + '.'   + java_minor_version|string) }} {% endraw %}
```

If you need to preserve line breaks use the following approach:

```yaml {% raw %}
- name: Warn on unsupported platform
  fail:
    msg: |
      This role does not support '{{ ansible_os_family }}' platform.
        Please contact support@lean-delivery.com {% endraw %}
```
---

## Booleans

Ansible supports many different ways to specify a boolean value: `True/False`, `true/false`, `yes/no`, `1/0`. We prefer to avoid mixing of various styles and stick to one : `true/false`.

```yaml
# bad
- name: start nginx
  service:
    name: nginx
    state: started
    enabled: yes
  become: yes
 
# good
- name: start nginx
  service:
    name: nginx
    state: started
    enabled: true
  become: true
```

### Why?
The main reasoning behind this is that Python have same convention for boolean values. 

---

## Colon spacing

Use only one space after the colon when designating a key value pair

```yaml
# bad
- name : start nginx
  service:
    name    : nginx
    state   : started
    enabled : true
  become : true


# good
- name: start nginx
  service:
    name: nginx
    state: started
    enabled: true
  become: true
```

### Why?

It's easier to edit avoiding extra aligning efforts for colons.

---

## Key-Value pairs

Ansible allows two types of YAML syntax:
* legacy key=value style
* structured map style

**Always use the map syntax,** regardless of how many pairs exist in the map.

```yaml
# bad
- name: Create configuration file with parameters
  file: path=/etc/sensu/conf.d/checks state=directory mode=0755 owner=sensu group=sensu
  become: true
  
- name: Copy configuration file
  copy: dest=/etc/sensu/conf.d/checks/ src=checks/check-memory.json
  become: true
  
# good
- name: Create configuration file with parameters
  file:
    path: /etc/sensu/conf.d/checks
    state: directory
    group: sensu
    mode: 0755
    owner: sensu
  become: true
  
- name: Copy configuration file
  copy:
    dest: /etc/sensu/conf.d/checks/
    src: checks/check-memory.json
  become: true
```

### Why?
{: .no_toc }

The legacy key=value syntax is used on the command line for adhoc commands. The use of this style within playbooks and roles is a bad practice making playbooks harder to read and support. 

Structured map style makes code more compact in terms of line length and that makes code more easy to read. Each module parameter is places on its own line making possible to comment parameters independently. YAML syntax highlighting works better for this format allowing key/value detection, constants highlighting etc.

---

## Variable Names

Use `snake_case` for variable names:

```yaml
# bad
- name: set some facts
  set_fact:
    myBoolean: true
    myint: 20
    MY_STRING: test

# good
- name: set some facts
  set_fact:
    my_boolean: true
    my_int: 20
    my_string: test
```

### Why?
{: .no_toc }

Ansible uses `snake_case` for module names and parameters so it makes sense to extend this convention to variable names to unify style.
