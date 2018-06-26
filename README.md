# devgateway.postgres-replication

Set up a two-node Postgres replica set.

## Required Variables

### `pgrs_master`

Replica set master host. It will also be added to `.pgpass` file.

### `pgrs_slave`

Replica set slave host. It will also be added to `.pgpass` file.

### `pgrs_user`

A dictionary defining the user to run replication as. See `pgrs_hba_defaults` below for the format
description.

## Optional Variables

### `pgrs_archive_dir`

The directory where WALs are stored.

Default: ``` /var/lib/pgsql/archive ```

### `pgrs_bin_dir`

Directory where `pg_ctl` binary resides.

Default: ``` /usr/bin ```

### `pgrs_data_dir`

Postgres data directory.

Default: ``` /var/lib/pgsql/data ```

### `pgrs_hba_defaults`

A list of dictionaries representing users of host-based authentication in Postgres. Fields are the
same as in `pg_hba.conf` file, and **address** is optional.

Default:

* {'type': 'local', 'name': 'all', 'database': 'all', 'method': 'trust'}
* {'type': 'host', 'name': 'all', 'database': 'all', 'address': '127.0.0.1/32', 'method': 'trust'}
* {'type': 'host', 'name': 'all', 'database': 'all', 'address': '::1/128', 'method': 'trust'}

### `pgrs_pg_settings`

A dictionary of directives for `postgresql.conf`.

Default:

* **listen\_addresses**: ``` * ```
* **wal\_level**: ``` hot_standby ```
* **synchronous\_commit**: *True*
* **archive\_mode**: *True*
* **archive\_command**: ``` cp %p {{ pgrs_archive_dir }}/%f ```
* **max\_wal\_senders**: 5
* **wal\_keep\_segments**: 32
* **hot\_standby**: *True*
* **wal\_sender\_timeout**: 5000
* **wal\_receiver\_status\_interval**: 2
* **max\_standby\_streaming\_delay**: -1
* **max\_standby\_archive\_delay**: -1
* **restart\_after\_crash**: *False*
* **hot\_standby\_feedback**: *True*


### `pgrs_pgpass_hosts`

List of hosts to be added to `.pgpass` file for user **postgres**. Normally, you don't want to
override this, unless you're adding a virtual IP address for the replica set current master.

Default:

* ``` {{ pgrs_slave }} ```
* ``` {{ pgrs_master }} ```


### `pgrs_recovery_settings`

A dictionary of directives for `recovery.conf` on the slave.

Default:

* **standby\_mode**: *True*
* **primary\_conninfo**: ``` host={{ pgrs_master }} user={{ pgrs_user.name }} application_name={{ pgrs_slave }}-slave ```
* **restore\_command**: ``` cp {{ pgrs_archive_dir }}/%f %p ```
* **recovery\_target\_timeline**: ``` latest ```


### `pgrs_replica_set_master`

Hostname or IP address of the current master. In failover HA configuration, this is the virtual IP
address.

Default: ``` {{ pgrs_master }} ```

### `pgrs_service`

Systemd/init name of the service for Postgres.

Default: ``` postgresql ```

### `pgrs_slave_config`

Path to config file for `pgrs_slave_script`.

Default: ``` /etc/pg_slave_init.conf ```

### `pgrs_slave_script`

Install path of the script which clear Postgres data on slave and performs a base backup.

Default: ``` /usr/local/bin/pg_slave_init.sh ```

## Dependencies

* [devgateway.vendor-postgres](https://github.com/devgateway/ansible-vendor-postgres)

## Playbook Example

    ---
    - hosts:
        - alpha
        - bravo
      tasks:
        - name: Configure PG cluster
          include_role:
            name: devgateway.postgres-replication
          vars:
            pgrs_master: alpha
            pgrs_slave: bravo
            pgrs_data_dir: /var/lib/pgsql/9.4/data
            pgrs_bin_dir: /usr/pgsql-9.4/bin
            pgrs_archive_dir: /var/lib/pgsql/9.4/archive
            pgrs_service: postgresql-9.4
            pgrs_user:
              type: host
              name: replication
              database: replication
              address: samenet
              method: md5
              password: toor


## License

GPLv3+

## Author Information

Copyright 2018, Development Gateway
