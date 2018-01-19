# Prometheus Exporters

Set up common prometheus exporter configurations

## Install to requirements.yml

```
- src: git+git@github.com:smartlogic/ansible-role-prometheus-exporters
  name: prometheus-exporters
  version: 0.1.0
```

## Requirements

### For postgres_exporter

Set up permission for the prometheus user (by default) to have access to the necessary stats

```sql
CREATE USER prometheus;
ALTER USER prometheus SET SEARCH_PATH TO prometheus,pg_catalog;

-- If deploying as non-superuser (for example in AWS RDS)
-- GRANT prometheus TO :MASTER_USER;
CREATE SCHEMA prometheus AUTHORIZATION prometheus;

CREATE VIEW prometheus.pg_stat_activity
AS
  SELECT * from pg_catalog.pg_stat_activity;

GRANT SELECT ON prometheus.pg_stat_activity TO prometheus;

CREATE VIEW prometheus.pg_stat_replication AS
  SELECT * from pg_catalog.pg_stat_replication;

GRANT SELECT ON prometheus.pg_stat_replication TO prometheus;
```

## Role Variables

- `node_exporter_version` - Which version of node exporter to download
- `node_exporter_checksum` - The checksum for the version of node exporter
- `redis_exporter_version` - Which version of redis_exporter to download
- `redis_exporter_checksum` - The checksum for the version of redis_exporter
- `postgres_exporter_version` - Which version of postgres_exporter to download
- `postgres_exporter_checksum` - The checksum for the version of postgres_exporter
- `postgres_exporter_connection_host` - The host to connect to
  - Default: `/var/run/postgresql/` to use the default postgres socket
- `postgres_exporter_connection_ssl_mode` - If to use ssl when connecting
  - Default: `disable` - because the default is socket connection
- `postgres_exporter_connection_user` -
  - Default: `prometheus` - the connecting user (uses identity be default)

## Dependencies

None

## Example Playbook

Node exporter only:

```yaml
- hosts: servers
  roles:
    - { role: prometheus, actions: ["node_exporter"] }
```

Redis exporter only:

```yaml
- hosts: servers
  roles:
    - { role: prometheus, actions: ["redis_exporter"] }
```

Postgres exporter only:

```yaml
- hosts: servers
  roles:
    - { role: prometheus, actions: ["postgres_exporter"] }
```

## License

MIT

## Author Information

SmartLogic. https://smartlogic.io
