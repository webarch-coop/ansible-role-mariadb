# Ansible Debian MariaDB Role 

This repository contains an Ansible role for installing [MariaDB](https://mariadb.org/) on Debian Stertch and Buster servers.

See the [defaults/main.yml](defaults/main.yml) for the options that can be set.

To use this role you need to use Ansible Galaxy to install it into another repository under `galaxy/roles/mariadb` by adding a `requirements.yml` file in that repo that contains:

```yml
---
- name: mariadb
  src: https://git.coop/webarch/mariadb.git
  version: master
  scm: git
```

And a `ansible.cfg` that contains:

```
[defaults]
retry_files_enabled = False
pipelining = True
inventory = hosts.yml
roles_path = galaxy/roles

```

And a `.gitignore` containing:

```
roles/galaxy
```

To pull this repo in run:

```bash
ansible-galaxy install -r requirements.yml --force 
```

The other repo should also contain a `mariadb.yml` file that contains:

```yml
---
- name: Install MariaDB
  become: yes

  hosts:
    - mariadb_servers

  roles:
    - mariadb
```

And a `hosts.yml` file that contains lists of servers, for example:

```yml
---
all:
  children:
    mariadb_servers:
      hosts:
        host3.example.org:
        host4.example.org:
        cloud.example.com:
        cloud.example.org:
        cloud.example.net:
```

Then it can be run as follows:

```bash
ansible-playbook mariadb.yml 
```

## Creating multiple users and databases

You can call the `mariadb_user.yml` tasks multiple times, for example:

```yml
- name: Create database and user for WordPress
  include_role: mariadd
    tasks_from: mariadb_user.yml
  vars: 
    mariadb_database: wordpress
    mariadb_username: wordpress

- debug:
    msg: "The MariaDB password for WordPress is: {{ mariadb_password }}"

- name: Create database and user for Matomo
  include_role: mariadd
    tasks_from: mariadb_user.yml
  vars:
    mariadb_database: matomo
    mariadb_username: matomo

- debug:
    msg: "The MariaDB password for Matomo is: {{ mariadb_password }}"
```

Note that the `mariadb_password` variable will contain the password for
the last user created.

## TODO

* Check that the mariadb_username and mariadb_database are lowercase and contain no punctuation or white space 
* Add additional optional `mariadb_` variables for values in `templates/50-server.cnf.j2`
* Consider adding the ability to create multiple database users and databases, reading these from YAML dicts, for example:
```yml
  vars:
    mariadb_databases_present:
      - wordpress_prod
      - wordpress_stage
    mariadb_databases_absent:
      - drupal_prod
      - drupal_stage
    mariadb_users_present:
      - name: wordpress
        priv:
          - 'wordpress_prod.*:ALL'
          - 'wordpress_stage.*:ALL'
      
```
