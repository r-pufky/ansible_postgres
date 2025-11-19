# PostgresQL
Manage Postgres databases.

## Requirements
[supported platforms](https://github.com/r-pufky/ansible_postgres/blob/main/meta/main.yml)

`psycopg2` is required on ansible controller for community.postgresql.

## Role Variables
[defaults](https://github.com/r-pufky/ansible_postgres/tree/main/defaults/main)

### Ports
All ports and protocols have been defined for the role.

[defaults/ports.yml](https://github.com/r-pufky/ansible_postgres/blob/main/defaults/main/ports.yml)

## Dependencies
**galaxy-ng** roles cannot be used independently. Part of
[r_pufky.srv](https://github.com/r-pufky/ansible_collection_srv) collection.

## Example Playbook
Read defaults documentation. All options are supported. Multiple versions of
Postgres may be managed by applying the Role separately for each version.

[Additional documentation](http://r-pufky.github.io/r-pufky/docs/service/postgres).

### Setup Postgres, create user, create database, enable backups.
host_vars/db.example.com/vars/postgres.yml
``` yaml
postgres_cfg_databases_backups_enable: true
postgres_cfg_users:
  - name: 'example'
    host: 'localhost'
    password: '{{ vault_db_test_password }}'
    state: 'present'
    append_privs: false
    encrypted: true
    extensions:
      priv:
        - login_db: 'postgres'
          privs: 'ALL'
          type: 'database'
        - login_db: 'some_app'
          privs: 'ALL'
          type: 'database'
postgres_cfg_databases:
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
postgres_cfg_databases_backups_enable: true
postgres_cfg_users:
  - name: 'example'
    host: 'localhost'
    password: '{{ vault_db_test_password }}'
    state: 'present'
    append_privs: false
    encrypted: true
    extensions:
      database_owner:
        - 'some_app'
postgres_cfg_databases:
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
Apply role for each specific instance. Postgres releases outside of the role
releases version may have limited suppport.

See documentation.

Please submit a bug if a new version of Postgres has been released and updated
default configuration variables are needed.

## Development
Configure [environment](https://r-pufky.github.io/ansible_collection_docs/ansible/environment)

Run all unit tests:
``` bash
molecule test --all
```

### Releases
Release format: **{OS}-{SERVICE}-{ROLE}**

Each type inherits the versioning system used; defaulting to schematic
versioning.

`12.0.0-2.0.3-1.0.0`

* 12.0.0 - Debian 12 (bookworm).
* 2.0.3 - Service/app version.
* 1.0.0 - Role version.

Releases are branched on Debian releases:

* **[13.x.x](https://github.com/r-pufky/ansible_postgres)**: 13 Trixie.
* **[12.x.x](https://github.com/r-pufky/ansible_postgres/tree/12.x)**: 12 Bookworm.

## Issues
Create a bug and provide as much information as possible.

Associate pull requests with a submitted bug.

## License
[AGPL-3.0 License](https://www.tldrlegal.com/license/gnu-affero-general-public-license-v3-agpl-3-0)
 [(direct link)](https://github.com/r-pufky/ansible_postgres/blob/main/LICENSE)

## Author Information
PGP Fingerprint: [466EEC2B67516C7117C85CE3A0BC35D16698BAB9](https://keys.openpgp.org/vks/v1/by-fingerprint/466EEC2B67516C7117C85CE3A0BC35D16698BAB9)
| [github gist](https://gist.github.com/r-pufky/a8df36977c55b5bb20829267c4c49d22)
