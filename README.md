# Webarchitects Ansible Debian MariaDB Role 

[![pipeline status](https://git.coop/webarch/mariadb/badges/master/pipeline.svg)](https://git.coop/webarch/mariadb/-/commits/master)

This repository contains an Ansible role for installing and configuring [MariaDB](https://mariadb.org/) on Debian servers.

Versions of this role including and prior to [version 1.9.1](https://git.coop/webarch/mariadb/-/tree/1.9.1) require Ansible 2.9 and use the command and shell modules for many tasks, version 2.0.0 onwards requires Ansible 2.10 and uses the [community.mysql collection](https://docs.ansible.com/ansible/latest/collections/community/mysql/). 

The `community.mysql` collection modules can be installed into `~/.ansible/collections/ansible_collections` like this:

```bash
ansible-galaxy collection install community.mysql
```

This role can be used to [switch the root users authentication plugin from `unix_socket` to `mysql_native_password`](tasks/mariadb_root_password.yml) and [back](tasks/info_socket.yml) and it also imports the [sys schema](https://github.com/webarch-coop/mariadb-sys), [updates the timezone data](tasks/tz.yml) when needed and sets [some systemd defaults](templates/mariadb.conf.j2).

This role adds [a script](templates/mariadb_root.fact.j2) to `/etc/ansible/facts.d` which 

## Defaults

See also the [defaults/main.yml](defaults/main.yml) file.

| Variable name                          | Default value                                      | Comment                                                                                                                       |
|----------------------------------------|----------------------------------------------------|-------------------------------------------------------------------------------------------------------------------------------|
| `mariadb`                              | `true`                                             | Set `mariadb` to false to prevent any tasks in this role being run                                                            |
| `mariadb_host`                         | `localhost`                                        | Note that this roles hasn't been tested with hosts other than `localhost`                                                     |
| `mariadb_port`                         | `3306`                                             | The default MariaDB port                                                                                                      |
| `mariadb_path`                         | `/usr/bin/mariadb`                                 | The existance of the `mariadb_path` is used as a test for generating the `local_facts`                                        |
| `mariadb_socket`                       | `/var/run/mysqld/mysqld.sock`                      | The path to the MariaDB scoket                                                                                                |
| `mariadb_sys_schema`                   | `true`                                             | If `mariadb_sys_schema` is true then the sys schema is imported from [this repo](https://github.com/webarch-coop/mariadb-sys) |
| `mariadb_time_zone_import`             | `true`                                             | If `mariadb_time_zone_import` is true then the  time zone tables when they have been updated                                  |
| `mariadb_systemd_no_new_privileges`    | `true`                                             | Set systemd `NoNewPrivileges` to true for MariaDB                                                                             |
| `mariadb_systemd_private_network`      | `true`                                             | Set systemd `PrivateNetwork` to true for MariaDB                                                                              |
| `mariadb_systemd_private_tmp`          | `true`                                             | Set systemd `PrivateTmp` to true for MariaDB                                                                                  |
| `mariadb_join_buffer_size`             | `8M`                                               |                                                                                                                               |
| `mariadb_open_files_limit`             | `122880`                                           |                                                                                                                               |
| `mariadb_table_open_cache`             | `4000`                                             |                                                                                                                               |
| `mariadb_tmp_table_size`               | `32M`                                              |                                                                                                                               |
| `mariadb_max_heap_table_size`          | `"{{ mariadb_tmp_table_size }}"`                   |                                                                                                                               |
| `mariadb_max_allowed_packet`           | `64M`                                              |                                                                                                                               |
| `mariadb_key_buffer_size`              | `16M`                                              |                                                                                                                               |
| `mariadb_max_connections`              | `80`                                               |                                                                                                                               |
| `max_user_connections`                 | `0`                                                |                                                                                                                               |
| `mariadb_table_cache`                  | `64`                                               |                                                                                                                               |
| `mariadb_thread_concurrency`           | `10`                                               |                                                                                                                               |
| `mariadb_innodb_buffer_pool_size`      | `1G`                                               |                                                                                                                               |
| `mariadb_innodb_log_file_size`         | `256M`                                             |                                                                                                                               |
| `mariadb_innodb_buffer_pool_instances` | `1`                                                |                                                                                                                               |
| `mariadb_query_cache_type`             | `0`                                                |                                                                                                                               |
| `mariadb_query_cache_limit`            | `0`                                                |                                                                                                                               |
| `mariadb_query_cache_size`             | `0`                                                |                                                                                                                               |
| `mariadb_root_auth`                    |                                                    |                                                                                                                               |
| `mariadb_username`                     |                                                    |                                                                                                                               |
| `mariadb_priv`                         |                                                    |                                                                                                                               |
| `mariadb_database`                     |                                                    |                                                                                                                               |
| `mariadb_password`                     |                                                    |                                                                                                                               |
| `mariadb_mycnf`                        |                                                    |                                                                                                                               |

## Creating users and databases

You can call the `mariadb_user.yml` tasks multiple times, for example:

```yml
- name: Create database and user for WordPress
  include_role: mariadb
    tasks_from: mariadb_user.yml
  vars: 
    mariadb_database: wordpress
    mariadb_username: wordpress

- debug:
    msg: "The MariaDB password for WordPress is: {{ mariadb_password }}"

- name: Create database and user for Matomo
  include_role: mariadb
    tasks_from: mariadb_user.yml
  vars:
    mariadb_database: matomo
    mariadb_username: matomo

- debug:
    msg: "The MariaDB password for Matomo is: {{ mariadb_password }}"
```

Note that the `mariadb_password` variable will contain the password for the last user created.
