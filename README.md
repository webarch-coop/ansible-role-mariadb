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

Set the `mariadb` variable to `true` for the tasks in this role to be run, it defaults to `false`.

### mariadb_config

An optional list of dictionaries which each require a `path` and a `state`, the `path` is the MariaDB configuration file path and the `state` specifies the state of the file, `absent` for removal, `edited` for an existing file to be edited using the [community.general.ini_file module](https://docs.ansible.com/ansible/latest/collections/community/general/ini_file_module.html), `present` for the file to be edited if it exists and `templated` if it doesn't, `templated` uses the [ansible.builtin.template module](https://docs.ansible.com/ansible/latest/collections/ansible/builtin/template_module.html) to create or clobber the file.

Optional variables are `conf` for a YAML dictionary representing the file configuration in the same format as provided by the [the JC ini parser](https://kellyjonbrazil.github.io/jc/docs/parsers/ini), `group` for the file group, `mode` for the octal mode of the file, `woner` for the owner of the file and `name` for a description of the file, for example:

```yaml
mariadb_config:
  - name: MariaDB client configuration
    path: /etc/mysql/mariadb.conf.d/50-client.cnf
    state: edited
    conf:
      client:
        default_character_set: utf8mb4
```

### mariadb_mysqltuner

A boolean, `mariadb_mysqltuner` defaults to `true` and results in [MySQLTuner](https://github.com/major/MySQLTuner-perl) being installed using a Debian package or from GitHub depending on the version specified using `mariadb_mysqltuner_version`.

### mariadb_mysqltuner_version

A version number for MySQLTuner, `mariadb_mysqltuner_version` defaults to `1.9.9` the version provided by [Debian Bookworm](https://packages.debian.org/bookworm/mysqltuner). If `mariadb_mysqltuner_version` is set to `latest` then the [versions available from GitHub](https://github.com/major/MySQLTuner-perl/releases) are checked and the latest release is installed.

### mariadb_pkgs

A list of Debian packages to install, the default value for `mariadb_pkgs`:

```yaml
mariadb_pkgs:
  - git
  - jo
  - mariadb-client
  - mariadb-server
  - mycli
  - pwgen
  - python3-mysqldb
```

### mariadb_socket

The path to the MariaDB socket.

### mariadb_systemd_units

A list to be use with the [systemd role](https://git.coop/webarch/systemd) as `systemd_units`, by default this role sets `PrivateNetwork` to `true`, set this to `false` if you need to connect to the server using `127.0.0.1` / TCP/IP in addition as `localhost`, which uses the socket.

The default value for `mariadb_systemd_units`:

```yaml
mariadb_systemd_units:
  - name: mariadb
    state: enabled
    files:
      - path: /etc/systemd/system/mariadb.service.d/mariadb.conf
        conf:
          Service:
            NoNewPrivileges: "true"
            PrivateNetwork: "true"
            PrivateTmp: "true"
            LimitNOFILE: "122880"
```

### mariadb_sys_schema

A boolean which defaults to `false`, [MariaDB 10.6.0 and greater](https://mariadb.com/kb/en/sys-schema/) provides a Sys Schema, Debian Bookworm provides [10.11.2](https://packages.debian.org/bookworm/mariadb-server), with older versions of MariaDB this role can optionally install version [1.5.3](https://git.coop/webarch/mariadb-sys/-/releases/v1.5.3) of [this MariaDB sys schema](https://git.coop/webarch/mariadb-sys).

### mariadb_time_zone_import

A boolean, which defaults to `true`. which results in `mysql_tzinfo_to_sql` being used to convert `/usr/share/zoneinfo` into SQL which is then imported into the `mysql` database.

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
