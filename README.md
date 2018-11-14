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
  # * If the mariadb_username is not set no user account or database will be 
  #   created 
  #
  # * If the mariadb_username is set but the mariadb_database name is not set 
  #   then the mariadb_username will be used as the mariadb_database name and 
  #   then the database will be created, if it didn't already exist
  #
  # * If the mariadb_password is set then mariadb_username will be created or
  #   updated with the mariadb_password and this will also be written to  
  #   ~/.my.cnf if the mariadb_username matches a username in /etc/passwd or 
  #   /root/.username.my.cnf if it doesn't
  #
  #   If the mariadb_password is not set and the ~/.my.cnf or 
  #   /root/.username.my.cnf files exist then the mariadb_password will be 
  #   read from the file and the mariadb_username login updated to match
  #
  #   If the mariadb_password is not set and the ~/.my.cnf or
  #   /root/.username.my.cnf files don't exist then a random mariadb_password
  #   will be generated and written to ~/.my.cnf or /root/.username.my.cnf 
  #   and the mariadb_username login updated to match
  #
  # * If the value of mariadb_root_password is "set" and if /root/.my.cnf 
  #   exists then mariadb_root_password will be read from the file, if the 
  #   file doesn't exist then a new password will be generated and the 
  #   mariadb_root_password variable will be set
  #   
  #   Then the mariadb_root_password is written to the database and 
  #   /root/.my.cnf 
  #
  #vars:
  #  - mariadb_root_password: set
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

* Check that the mariadb_username and mariadb_database are lowercase and contain no punctuation or white space 
* Add additional optional `mariadb_` variables for values in `templates/50-server.cnf.j2`
* Consider adding the ability to add multiple database users and databases 
