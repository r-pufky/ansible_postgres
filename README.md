# PostgresQL
Manage Postgres databases.

## Requirements
[supported platforms](https://github.com/r-pufky/ansible_postgres/blob/main/meta/main.yml)

[collections/roles](https://github.com/r-pufky/ansible_postgres/blob/main/meta/requirements.yml)

## Role Variables
[defaults](https://github.com/r-pufky/ansible_postgres/tree/main/defaults/main)

### Ports
All ports and protocols have been defined for the role.

[defaults/ports.yml](https://github.com/r-pufky/ansible_postgres/blob/main/defaults/main/ports.yml)

## Dependencies
Part of the [r_pufky.srv](https://github.com/r-pufky/ansible_collection_srv)
collection.

## Example Playbook
Read defaults documentation. All options are supported. Multiple versions of
Postgres may be managed by applying the Role separately for each version.

### Setup Postgres, create user, create database, enable backups.
host_vars/db.example.com/vars/postgres.yml
``` yaml
postgres_databases_backups_enable: true
postgres_users:
  - name: 'example'
    host: 'localhost'
    password: '{{ vault_db_test_password }}'
    state: 'present'
    append_privs: false
    encrypted: true
    extensions:
      priv:
        - database: 'postgres'
          privs: 'ALL'
          type: 'database'
        - database: 'some_app'
          privs: 'ALL'
          type: 'database'
postgres_databases:
  - name: 'some_app'
    owner: 'example'
    extensions:
      db_import:
        enable: true
```

site.yml
``` yaml
- name: 'db server'
  hosts: 'db.example.com'
  become: true
  roles:
     - 'r_pufky.srv.postgres'
```

### Setup Postgres and import database with ownership on database creation.
Import is idempotent -- database is only imported when the database is created.
Re-import will require database deletion. All user extensions are applied
every role application.

host_vars/db.example.com/vars/postgres.yml
``` yaml
postgres_databases_backups_enable: true
postgres_users:
  - name: 'example'
    host: 'localhost'
    password: '{{ vault_db_test_password }}'
    state: 'present'
    append_privs: false
    encrypted: true
    extensions:
      database_owner:
        - 'some_app'
postgres_databases:
  - name: 'some_app'
    owner: 'example'
    extensions:
      db_import:
        enable: true
        source: 'host_vars/db.example.com/files/skeleton_default.sql'
```

site.yml
``` yaml
- name: 'db server'
  hosts: 'db.example.com'
  become: true
  roles:
     - 'r_pufky.srv.postgres'
```

### Deploying versions with different configuration settings.
Role will automatically parse `postgres_config` variables and render them
correctly to `postgresql.conf`. See documentation. Just **remove** or **add**
variables for the version of Postgres being installed.

Please submit a bug if a new version of Postgres has been releases and updated
default configuration variables are needed.

host_vars/db.example.com/vars/postgres.yml
``` yaml
postgres_config:
  ...
  some_deprecated_option: 'some value'
  some_new_option: 'some value'
  ...
postgres_users:
  - name: 'example'
    host: 'localhost'
    password: '{{ vault_db_test_password }}'
    state: 'present'
    append_privs: false
    encrypted: true
    extensions:
      priv:
        - database: 'postgres'
          privs: 'ALL'
          type: 'database'
        - database: 'some_app'
          privs: 'ALL'
          type: 'database'
postgres_databases:
  - name: 'some_app'
    owner: 'example'
    extensions:
      db_import:
        enable: true
        source: 'host_vars/db.example.com/files/skeleton_default.sql'
```

site.yml
``` yaml
- name: 'db server'
  hosts: 'db.example.com'
  become: true
  roles:
     - 'r_pufky.srv.postgres'
```

## Development
Configure [environment](https://github.com/r-pufky/ansible_collection_srv/blob/main/docs/dev/environment/README.md)

Run all unit tests:
``` bash
molecule test --all
```

## Issues
Create a bug and provide as much information as possible.

Associate pull requests with a submitted bug.

## License
[AGPL-3.0 License](https://www.tldrlegal.com/license/gnu-affero-general-public-license-v3-agpl-3-0)
 [(direct link)](https://github.com/r-pufky/ansible_postgres/blob/main/LICENSE)

## Author Information
PGP Fingerprint: [466EEC2B67516C7117C85CE3A0BC35D16698BAB9](https://keys.openpgp.org/vks/v1/by-fingerprint/466EEC2B67516C7117C85CE3A0BC35D16698BAB9)
| [github gist](https://gist.github.com/r-pufky/a8df36977c55b5bb20829267c4c49d22)
