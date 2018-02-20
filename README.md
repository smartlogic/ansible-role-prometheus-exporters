# Prometheus Exporters

Set up common prometheus exporter configurations

## Install to requirements.yml

```
- src: git+git@github.com:smartlogic/ansible-role-prometheus-exporters
  name: prometheus-exporters
  version: 0.5.0
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

### Firewall Ports

These ports should be opened to the prometheus server based on the enabled exporters:

| Exporter      | Port  |
| ------------- | ----- |
| node          | 9100  |
| statsd        | 9102  |
| elasticsearch | 9108  |
| blackbox      | 9115  |
| redis         | 9121  |
| postgres      | 9187  |

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
- `blackbox_exporter_version` - Which version of blackbox_exporter to download
- `blackbox_exporter_checksum` - The checksum for the version of blackbox_exporter
- `statsd_exporter_version` - Which version of statsd_exporter to download
- `statsd_exporter_checksum` - The checksum for the version of statsd_exporter
- `statsd_mapping_file` - The file to use for statsd mapping to prometheus metrics
  - Default: an empty mappings file
- `elasticsearch_exporter_version` - Which version of elasticsearch_exporter to download
- `elasticsearch_exporter_checksum` - The checksum for the version of elasticsearch_exporter

## Dependencies

None

## Example Configuration

If using the statsd_exporter action:

```yaml
statsd_mapping_file: "{{ playbook_dir }}/files/statsd_mapping.yml"
```

## Example Playbook

Node exporter only:

```yaml
- hosts: servers
  roles:
    - { role: prometheus-exporters, actions: ["node_exporter"] }
```

Redis exporter only:

```yaml
- hosts: servers
  roles:
    - { role: prometheus-exporters, actions: ["redis_exporter"] }
```

Postgres exporter only:

```yaml
- hosts: servers
  roles:
    - { role: prometheus-exporters, actions: ["postgres_exporter"] }
```

Blackbox exporter only:

```yaml
- hosts: servers
  roles:
    - { role: prometheus-exporters, actions: ["blackbox_exporter"] }
```

Statsd exporter only:

```yaml
- hosts: servers
  roles:
    - { role: prometheus-exporters, actions: ["statsd_exporter"] }
```

Elasticsearch exporter only:

```yaml
- hosts: servers
  roles:
    - { role: prometheus-exporters, actions: ["elasticsearch_exporter"] }
```

## License

MIT

## Author Information

SmartLogic. https://smartlogic.io
