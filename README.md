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

  # ALL OF THESE VARIABLES ARE OPTIONAL
  # -----------------------------------
  #
  # If the following variables are not set then no tasks which depend on 
  # them will be run
  #
  #vars:
  #  - mariadb_root_password: set
  #  - mariadb_username:
  #  - mariadb_database:
  #  - mariadb_password: 
  #  - mariadb_character_set_client_handshake: FALSE
  #
  # NOTES ON THE VARIABLES
  # ----------------------
  #
  # * If the mariadb_root_password variable is not set then nothing will 
  #   be done to the root account login, which, by default, uses a 
  #   socket for logins and doesn't have a password set, generally you 
  #   will want to omit this option -- it is only for cases where 
  #   non-root users need to login to MariaDB as root that you need to 
  #   enable this
  #
  # * If the value of mariadb_root_password is "set" and if 
  #   /root/.my.cnf exists then mariadb_root_password will be read 
  #   from the file, if the file doesn't exist then a new password 
  #   will be generated and the mariadb_root_password variable will 
  #   be set
  # 
  # * The mariadb_root_password will be written to the database and 
  #   /root/.my.cnf 
  #
  # * If the mariadb_username is not set then no user account or database 
  #   will be created 
  #
  # * If the mariadb_username is set but the mariadb_database name is not set 
  #   then the mariadb_username will be used as the mariadb_database name 
  # 
  # * The mariadb_database will be created, if it didn't already exist
  # 
  # * If a system user account exists with the same name as the 
  #   mariadb_username then mariadb_mycnf is set to ~/.my.cnf
  #
  # * If there is no system user account name that matches the 
  #   mariadb_username then mariadb_mycnf is set to 
  #   /root/.mariadb_username.my.cnf
  #
  # * If mariadb_mycnf doesn't exists and mariadb_password is not set then
  #   a random password is generated for mariadb_password  
  #
  # * If mariadb_mycnf does exist then the mariadb_password is loaded from
  #   the file
  #
  # * The mariadb_password will be written to mariadb_mycnf
  #   and the mariadb_username database login updated to match

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
  include_tasks: galaxy/roles/mariadb/tasks/mariadb_user.yml
  vars: 
    mariadb_user: wordpress

- debug:
    msg: "The MariaDB password for WordPress is: {{ mariadb_password }}"

- name: Create database and user for Matomo
  include_tasks: galaxy/roles/mariadb/tasks/mariadb_user.yml
  vars:
    mariadb_user: matomo

- debug:
    msg: "The MariaDB password for Matomo is: {{ mariadb_password }}"
```

Note that the `mariadb_password` variable will only contain the password for the last user created and will be overwritten when another user is created.


## TODO

* Check that the mariadb_username and mariadb_database are lowercase and contain no punctuation or white space 
* Add additional optional `mariadb_` variables for values in `templates/50-server.cnf.j2`
* Consider adding the ability to create multiple database users and databases, reading these from a pair of YAML dicts, for example:
```yml
  vars:
    mariadb_users:
      wordpress:
        privs: 
         - wordpress.*:ALL
         - civicrm.*:ALL
      matomo:
         privs:
           - matomo.*:ALL
    maria_databases:
      - wordpress
      - matomo
```
    
