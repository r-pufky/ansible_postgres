# [PostgresQL][h]
Manage Postgres databases.

## [Requirements][i]
Requires [r_pufky.srv][g] galaxy-ng collection. See
[additional documentation][m] and [reference documentation][h] for
troubleshooting and config variables.

Install size: ~80MB

> **psycopg2** is required on ansible controller for community.postgresql.

> Tasks [potentially touching Network Mounted Filesystems][o] will be run as
> the task user and fallback to the service user. Manage these locations
> externally if these fail.

## Role Variables
Detailed variable use documented in defaults. See usage for role operation.

* [defaults][j] - User configurable options.

* [ports][k] - Ports are **not** managed (defined for external use).

## Usage
### WARNING
> Postgres will refuse to start with any [invalid configuration option][h],
> which include deprecated options and version mis-matched options. Default
> service does not log configuration errors on startup. This must be done
> manually to see what configuration issues are occurring.
>
> Database version upgrades are currently not supported and must be done
> manually (or export the database and import the database via the role).
>
> Multiple Postgres version deployments on the same machine will work but are
> unsupported.

``` bash
su - postgres
/usr/lib/postgresql/17/bin/postgres -d 3 \
-D /var/lib/postgresql/17/main \
-c config_file=/etc/postgresql/17/main/postgresql.conf
```

 Path                                       | Usage
 -------------------------------------------|-------
 /etc/postgres/{VERSION}/main               | Configuration files deployed here.
 /etc/postgres/{VERSION}/main/conf.d        | postgres_srv_conf_d always deployed here.
 /etc/postgres/{VERSION}/main/secure.conf.d | postgres_srv_secure_conf_d always deployed here.
 /etc/postgres/{VERSION}/scripts            | Role scripts.
 /var/lib/postgresql/{VERSION}              | Default database location.

### Feature Flags
Tasks are gated by feature flags and executed in the following order.

  Step | Flag                 | Notes
 ------|----------------------|-------
  1    | postgres_flg_install | Install required packages, users, etc.
  2    | postgres_flg_config  | Install user-defined config.
  3    | postgres_flg_backup  | Create scheduled backups?

### Example Playbooks

#### New Deployment
Postgres will be ready to use, allowing logins with **example**.

``` yaml
- name: 'Default Postgres install, example user, test1 DB, enabling backups.'
  ansible.builtin.include_role:
    name: 'r_pufky.srv.postgres'
  vars:
    postgres_flg_backup: true
    postgres_cfg_users:
      - name: 'example'
        host: 'localhost'
        password: 'example'
        state: 'present'
        append_privs: false
        encrypted: true
        extensions:
          # Alternatively, database_owner can use used to set privs.
          owner:
            - login_db: 'test1'
              obj_type: 'table'
            - login_db: 'test1'
              obj_type: 'sequence'
            - login_db: 'test1'
              obj_type: 'view'
          priv:
            - login_db: 'postgres'
              privs: 'ALL'
              type: 'database'
            - login_db: 'test1'
              privs: 'ALL'
              type: 'database'
    postgres_cfg_dbs:
      - name: 'test1'
        owner: 'example'
```

#### Create/Import Databases
Databases can be created automatically, optionally importing data from a SQL
backup if provided. Import is idempotent -- database is only imported when the
database is created. Re-import will require database deletion. All user
extensions are applied every role application.

Databases use community.postgresql.postgres_db parameters as dictionary keys
with default values matching specified parameters unless noted. Additional
attributes are fully defined. See [documentation][j].

Users use community.postgresql.postgres_user parameters as dictionary keys with
default values matching specified parameters unless noted. See
[documentation][j].

``` yaml
- name: 'Default Postgres insall, importing two databases.'
  ansible.builtin.include_role:
    name: 'r_pufky.srv.postgres'
  vars:
    postgres_cfg_users:
      - name: 'example'
        host: 'localhost'
        password: 'example'
        state: 'present'
        append_privs: false
        encrypted: true
        extensions:
          # Alternatively, owner/priv can be used to set privs.
          database_owner:
            - 'auto_import'
            - 'specify_sql_dump'
    postgres_cfg_dbs:
      - name: 'auto_import'
        owner: 'example'
        extensions:
          # Will automatically import the latest dump from schedule backups if
          # database is being created.
          db_import:
            enable: true
      - name: 'specify_sql_dump'
        owner: 'example'
        extensions:
          # Import specific dump on database creation.
          db_import:
            enable: true
            source: 'host_vars/db.example.com/files/custom_dump.sql'
```

#### Instance Configuration
Configure the Postgres instance. conf.d, environment, pg_ctl.conf, pg_hba.conf,
pg_ident.conf, postgresql.conf, start.conf, and profile.d/postgres are all
sourced from the ansible controller and templated. Place private material in
secure.conf.d, which will automatically be locked down.

Postgres configuration is complex and nuanced. See [config settings][n] keeping
in mind where the [role places files](#usage).

Example custom configuration below. See [templated defaults][n] for more
examples.

Example pg_hba.conf
``` ini
# PostgreSQL Client Authentication Configuration File

# Database administrative login by Unix domain socket
local all postgres  peer
# local is for Unix domain socket connections only
local all all  peer
# IPv4 local connections
host all all 127.0.0.1/32 scram-sha-256
# IPv6 local connections
host all all ::1/128 scram-sha-256
# Allow replication connections from localhost, by a user with the replication privilege.
local replication all  peer
host replication all 127.0.0.1/32 scram-sha-256
host replication all ::1/128 scram-sha-256
```

Example pg_ctl.conf
``` ini
# Automatic pg_ctl configuration
pg_ctl_options = ''
```

Example pg_ident.conf
``` ini
# Ident mapping
local_all root postgres
```

Example postgres
``` ini
# Environment variables for postgres processes
export PGDATA='/var/lib/postgresql/17/main'
export PATH='$PATH:/usr/lib/postgresql/17/bin'
```

Example start.conf
``` ini
# Automatic startup configuration
auto
```

Example postgres.conf
``` ini
# Partial file listing. See templated examples for full file.
data_directory='/var/lib/postgresql/17/main'
hba_file='/etc/postgresql/17/main/pg_hba.conf'
ident_file='/etc/postgresql/17/main/pg_ident.conf'
external_pid_file='/var/run/postgresql/17-main.pid'
listen_addresses='localhost'
port=5432
max_connections=100
reserved_connections=0
superuser_reserved_connections=3
unix_socket_directories='/var/run/postgresql'
unix_socket_group=''
unix_socket_permissions='0777'
bonjour=false
```

``` yaml
- name: 'Deploy pre-configured instance.'
  ansible.builtin.include_role:
    name: 'r_pufky.srv.postgres'
  vars:
    postgres_flg_config: true
    postgres_flg_backup: true
    postgres_srv_conf_d: 'host_vars/db.example.com/conf.d'
    postgres_srv_secure_conf_d: 'host_vars/db.example.com/secure.conf.d'
    postgres_srv_environment: 'host_vars/db.example.com/environment'
    postgres_srv_pg_ctl: 'host_vars/db.example.com/pg_ctl.conf'
    postgres_srv_pg_hba: 'host_vars/db.example.com/pg_hba.conf'
    postgres_srv_pg_ident: 'host_vars/db.example.com/pg_ident.conf'
    postgres_srv_postgresql: 'host_vars/db.example.com/postgresql.conf'
    postgres_srv_start: 'host_vars/db.example.com/start.conf'
    postgres_srv_profile: 'host_vars/db.example.com/profile.d/postgres'
    postgres_cfg_users:
      - name: 'example'
        host: 'localhost'
        password: 'example'
        state: 'present'
        append_privs: false
        encrypted: true
        extensions:
          database_owner:
            - 'test1'
    postgres_cfg_dbs:
      - name: 'test1'
        owner: 'example'
          db_import:
            enable: true
            source: 'host_vars/db.example.com/files/custom_dump.sql'
```

## Development
Configure [environment][a].

``` bash
# Run all tests.
molecule test --all
```

### [Releases][b]

  Release | Debian | Ansible | Postgres | Notes
 ---------|--------|---------|----------|-------
  3.x.x   | 13     | 2.20    | v17      | Ansible 2.20, feature flags, and semantic versioning.
  2.x.x   | 13     | 2.18    | v17      | Migrate to Debian Trixie.
  1.x.x   | 12     | 2.18    | v17      | Implement data annotations.
  0.x.x   | 12     | 2.18    | v14      | Initial release.

## Issues
Create a bug and provide as much information as possible.

Associate pull requests with a submitted bug.

## License
[AGPL-3.0 License][c] | [direct link][f]

## Author Information
PGP: [466EEC2B67516C7117C85CE3A0BC35D16698BAB9][d] | [github gist][e]


[a]: https://r-pufky.github.io/ansible_docs
[b]: https://semver.org/spec/v2.0.0
[c]: https://www.tldrlegal.com/license/gnu-affero-general-public-license-v3-agpl-3-0
[d]: https://keys.openpgp.org/vks/v1/by-fingerprint/466EEC2B67516C7117C85CE3A0BC35D16698BAB9
[e]: https://gist.github.com/r-pufky/a8df36977c55b5bb20829267c4c49d22

[f]: https://github.com/r-pufky/ansible_postgres/blob/main/LICENSE
[g]: https://github.com/r-pufky/ansible_collection_srv
[h]: https://www.postgresql.org/docs/current/config-setting.html
[i]: https://github.com/r-pufky/ansible_postgres/blob/main/meta/main.yml
[j]: https://github.com/r-pufky/ansible_postgres/tree/main/defaults/main/main.yml
[k]: https://github.com/r-pufky/ansible_postgres/blob/main/defaults/main/ports.yml
[m]: http://r-pufky.github.io/docs/service/postgres
[n]: https://github.com/r-pufky/ansible_postgres/tree/main/templates/default
[o]: https://r-pufky.github.io/ansible_docs/best_practice/patterns/#network-mounts