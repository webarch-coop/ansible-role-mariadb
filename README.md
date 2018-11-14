# Ansible Debian MariaDB Role 

This repository contains an Ansible role for installing [MariaDB](https://mariadb.org/) on Debian servers.

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

  # All these variables are optional
  #
  # * If the username is not set no user account or database will be created
  #
  # * If the database name is not set the username will be used as the database
  #   name 
  #
  # * If the password is not set a random one will be generated and saved to
  #   ~/.my.cnf if the username matches a username in /etc/passwd or 
  #   /root/.username.my.cnf if it doesn't
  #
  #vars:
  #  - mariadb_username:
  #  - mariadb_database:
  #  - mariadb_password: 
  #  - mariadb_character_set_client_handshake: FALSE

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

## TODO

* Add additional optional `mariadb_` variables for values in `templates/50-server.cnf.j2`
* Consider adding the ability to add multiple database users and databases 
