# Webarchitects Ansible Debian MariaDB Role

[![pipeline status](https://git.coop/webarch/mariadb/badges/master/pipeline.svg)](https://git.coop/webarch/mariadb/-/commits/master)

This repository contains an Ansible role for installing and configuring [MariaDB](https://mariadb.org/) on Debian servers.

## Role versions

Version 3.0.0 and greater of this role provide the option to edit or template MariaDB configuration files and use YAML dictionaries for the specification, existing files are read using [the JC ini parser](https://kellyjonbrazil.github.io/jc/docs/parsers/ini) and edited using the [community.general.ini_file module](https://docs.ansible.com/ansible/latest/collections/community/general/ini_file_module.html) or created or clobbered using the [ansible.builtin.template module](https://docs.ansible.com/ansible/latest/collections/ansible/builtin/template_module.html).

The last 2.x version of this role [version 2.4.2](https://git.coop/webarch/mariadb/-/releases/2.4.2) is the last version that contains Ansible tasks to switch between password and socket authentication for the root user, all 3.0.0 versions assume socket authentication is used.

Versions of this role including and prior to [version 1.9.1](https://git.coop/webarch/mariadb/-/tree/1.9.1) require Ansible 2.9 and use the command and shell modules for many tasks, version 2.0.0 onwards requires Ansible 2.10 and uses the [community.mysql collection](https://docs.ansible.com/ansible/latest/collections/community/mysql/).

This role can be used to [switch the root users authentication plugin from `unix_socket` to `mysql_native_password`](tasks/mariadb_root_password.yml) and [back](tasks/info_socket.yml) and it also runs `mysql_upgrade`, [imports](tasks/sys.yml) the [sys schema](https://github.com/webarch-coop/mariadb-sys), [updates the timezone data](tasks/tz.yml) when needed and sets [some systemd defaults](templates/mariadb.conf.j2).

## Role variables

See the [defaults/main.yml](defaults/main.yml) file for the default variables, the [vars/main.yml](vars/main.yml) file for the preset variables and the [meta/argument_specs.yml](meta/argument_specs.yml) file for the variable specification.

### mariadb

### mariadb_config

### mariadb_mysqltuner

### mariadb_mysqltuner_version

### mariadb_pkgs

### mariadb_socket

### mariadb_systemd_units

### mariadb_sys_schema

## mariadb_time_zone_import

## Creating users and databases

You can call the `mariadb_user.yml` tasks multiple times, for example:

```yaml
- name: Create database and user for WordPress
  include_role:
    name: mariadb
    tasks_from: mariadb_user.yml
  vars:
    mariadb_database: wordpress
    mariadb_username: wordpress
    mariadb_priv:
      - ALTER
      - CREATE
      - DELETE
      - INSERT
      - SELECT
      - UPDATE

- debug:
    msg: "The MariaDB password for WordPress is: {{ mariadb_password }}"

- name: Create database and user for Matomo
  include_role:
    name: mariadb
    tasks_from: mariadb_user.yml
  vars:
    mariadb_database: matomo
    mariadb_username: matomo

- debug:
    msg: "The MariaDB password for Matomo is: {{ mariadb_password }}"

- name: Create database and user for Nextcloud
  include_role:
    name: mariadb
    tasks_from: mariadb_user.yml
  vars:
    mariadb_database: nextcloud
    mariadb_username: nextcloud

- debug:
    msg: "The MariaDB password for Nextcloud is: {{ mariadb_password }}"
```

Note that the `mariadb_password` variable will contain the password for the last user created.

## Repository

The primary URL of this repo is [`https://git.coop/webarch/mariadb`](https://git.coop/webarch/mariadb) however it is also [mirrored to GitHub](https://github.com/webarch-coop/ansible-role-mariadb) and [available via Ansible Galaxy](https://galaxy.ansible.com/chriscroome/mariadb).

If you use this role please use a tagged release, see [the release notes](https://git.coop/webarch/mariadb/-/releases).

## Copyright

Copyright 2018-2023 Chris Croome, &lt;[chris@webarchitects.co.uk](mailto:chris@webarchitects.co.uk)&gt;.

This role is released under [the same terms as Ansible itself](https://github.com/ansible/ansible/blob/devel/COPYING), the [GNU GPLv3](LICENSE).
