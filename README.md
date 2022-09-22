# Webarchitects Ansible Debian MariaDB Role 

[![pipeline status](https://git.coop/webarch/mariadb/badges/master/pipeline.svg)](https://git.coop/webarch/mariadb/-/commits/master)

This repository contains an Ansible role for installing and configuring [MariaDB](https://mariadb.org/) on Debian servers.

Versions of this role including and prior to [version 1.9.1](https://git.coop/webarch/mariadb/-/tree/1.9.1) require Ansible 2.9 and use the command and shell modules for many tasks, version 2.0.0 onwards requires Ansible 2.10 and uses the [community.mysql collection](https://docs.ansible.com/ansible/latest/collections/community/mysql/). 

The `community.mysql` collection modules can be installed into `~/.ansible/collections/ansible_collections` like this:

```bash
ansible-galaxy collection install community.mysql
```

This role can be used to [switch the root users authentication plugin from `unix_socket` to `mysql_native_password`](tasks/mariadb_root_password.yml) and [back](tasks/info_socket.yml) and it also runs `mysql_upgrade`, [imports](tasks/sys.yml) the [sys schema](https://github.com/webarch-coop/mariadb-sys), [updates the timezone data](tasks/tz.yml) when needed and sets [some systemd defaults](templates/mariadb.conf.j2).

This role adds [a script](templates/mariadb_root.fact.j2) to `/etc/ansible/facts.d` which results in the `ansible_local.mariadb_root.plugin` variable being generated with the value(s) of the root authentication plugins, `auth_socket`, `unix_socket` and / or `mysql_native_password`. 

The primary URL of this repo is [`https://git.coop/webarch/mariadb`](https://git.coop/webarch/mariadb) and this is where the [release notes](https://git.coop/webarch/debug/-/releases) are, it is also [mirrored to GitHub](https://github.com/webarch-coop/ansible-role-debug) and [available via Ansible Galaxy](https://galaxy.ansible.com/chriscroome/debug).

## Defaults

See also the [defaults/main.yml](defaults/main.yml) file.

| Variable name                          | Default value                    | Comment                                                                                                                                                                     |
|----------------------------------------|----------------------------------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `mariadb`                              | `true`                           | Set `mariadb` to false to prevent any tasks in this role being run                                                                                                          |
| `mariadb_host`                         | `localhost`                      | Note that this roles hasn't been tested with hosts other than `localhost`                                                                                                   |
| `mariadb_port`                         | `3306`                           | The default MariaDB port                                                                                                                                                    |
| `mariadb_path`                         | `/usr/bin/mariadb`               | The existance of the `mariadb_path` is used as a test for generating the `local_facts`                                                                                      |
| `mariadb_socket`                       | `/var/run/mysqld/mysqld.sock`    | The path to the MariaDB scoket                                                                                                                                              |
| `mariadb_mysqltuner`                   | `true`                           | Install [MySQLTuner](https://github.com/major/MySQLTuner-perl)                                                                                                              |
| `mariadb_mysqltuner_version`           | `latest`                         | Set `latest` or a version from [the releases page](https://github.com/major/MySQLTuner-perl/releases), eg `v1.9.9`                                                          |
| `mariadb_sys_schema`                   | `false`                          | If `mariadb_sys_schema` is true then the sys schema is imported from [this repo](https://github.com/webarch-coop/mariadb-sys)                                               |
| `mariadb_time_zone_import`             | `true`                           | If `mariadb_time_zone_import` is true then the  time zone tables when they have been updated                                                                                |
| `mariadb_systemd_no_new_privileges`    | `true`                           | Set systemd `NoNewPrivileges` to true for MariaDB                                                                                                                           |
| `mariadb_systemd_private_network`      | `true`                           | Set systemd `PrivateNetwork` to true for MariaDB                                                                                                                            |
| `mariadb_systemd_private_tmp`          | `true`                           | Set systemd `PrivateTmp` to true for MariaDB                                                                                                                                |
| `mariadb_join_buffer_size`             | `8M`                             |                                                                                                                                                                             |
| `mariadb_open_files_limit`             | `122880`                         |                                                                                                                                                                             |
| `mariadb_table_open_cache`             | `4000`                           |                                                                                                                                                                             |
| `mariadb_tmp_table_size`               | `32M`                            |                                                                                                                                                                             |
| `mariadb_max_heap_table_size`          | `"{{ mariadb_tmp_table_size }}"` |                                                                                                                                                                             |
| `mariadb_max_allowed_packet`           | `64M`                            |                                                                                                                                                                             |
| `mariadb_key_buffer_size`              | `16M`                            |                                                                                                                                                                             |
| `mariadb_max_connections`              | `80`                             |                                                                                                                                                                             |
| `max_user_connections`                 | `0`                              |                                                                                                                                                                             |
| `mariadb_table_cache`                  | `64`                             |                                                                                                                                                                             |
| `mariadb_thread_concurrency`           | `10`                             |                                                                                                                                                                             |
| `mariadb_innodb_buffer_pool_size`      | `1G`                             |                                                                                                                                                                             |
| `mariadb_innodb_log_file_size`         | `256M`                           |                                                                                                                                                                             |
| `mariadb_innodb_buffer_pool_instances` | `1`                              |                                                                                                                                                                             |
| `mariadb_query_cache_type`             | `0`                              |                                                                                                                                                                             |
| `mariadb_query_cache_limit`            | `0`                              |                                                                                                                                                                             |
| `mariadb_query_cache_size`             | `0`                              |                                                                                                                                                                             |
| `mariadb_root_auth`                    | `socket`                         | Set to `password` or `socket` to switch the root authentication plugin                                                                                                      |
| `mariadb_username`                     |                                  | Provide a `mariadb_username` to add a MariaDB user account                                                                                                                  |
| `mariadb_database`                     |                                  | If `mariadb_username` is set and `mariadb_database` is not set then the DB value will default to `mariadb_username`                                                         |
| `mariadb_password`                     |                                  | This variable is randomly generated and written to `/.my.cnf` or set to the value in `/.my.cnf` if it is present                                                            |
| `mariadb_priv`                         |                                  | An array of user `PRIVILEGES`, if `mariadb_priv` is not set it defaults to `ALL`                                                                                            |
| `mariadb_mycnf`                        |                                  | If a Linux user account exists that matches `mariadb_username` this will be set to `/home/{{ mariadb_username }}/.my.cnf` and if not `/root/{{ mariadb_username }}/.my.cnf` |

## Creating users and databases

You can call the `mariadb_user.yml` tasks multiple times, for example:

```yml
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
```

Note that the `mariadb_password` variable will contain the password for the last user created.
