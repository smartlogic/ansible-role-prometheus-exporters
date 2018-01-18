# Prometheus

Set up common prometheus configurations

## Install to requirements.yml

```
- src: git+git@github.com:smartlogic/ansible-role-prometheus
  name: prometheus
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

- `prometheus_version` - Which version of prometheus to download
- `prometheus_checksum` - The checksum for the version of prometheus
- `node_exporter_version` - Which version of node exporter to download
- `node_exporter_checksum` - The checksum for the version of node exporter
- `alertmanager_version` - Which version of alertmanager to download
- `alertmanager_checksum` - The checksum for the version of alertmanager
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
- `grafana_ini_file` - The file to use for `grafana.ini`
  - Default: `grafana.ini`
- `prometheus_config_file` - The file to use for `prometheus.yml`
  - Default: `prometheus.yml`
- `prometheus_alert_file` - The file to use for `alertmanager.yml`
  - Default: `alertmanager.yml`
- `alertmanager_enabled` - boolean - True for enabling the alertmanager, via systemctl
  - Default: `true`
- `prometheus` - Configure scrape files, alert rules, and alert templates

## Dependencies

None

## Example Configuration

```yaml
grafana_ini_file: "{{ playbook_dir }}/files/grafana.ini"
prometheus_config_file: "{{ playbook_dir }}/files/prometheus.yml"
prometheus_alert_file: "{{ playbook_dir }}/files/alertmanager.yml"

prometheus:
  jobs:
    - "{{ playbook_dir }}/file_config/project.json"
  alert:
    rules:
      - "{{ playbook_dir }}/alert_rules/rules.yml"
    templates:
      - "{{ playbook_dir }}/alert_templates/templte.tmpl"
```

## Example Playbook

Full setup:

```yaml
- hosts: servers
  roles:
    - { role: prometheus, action: "full" }
```

Node exporter only:

```yaml
- hosts: servers
  roles:
    - { role: prometheus, action: "node_exporter" }
```

Redis exporter only:

```yaml
- hosts: servers
  roles:
    - { role: prometheus, action: "redis_exporter" }
```

Postgres exporter only:

```yaml
- hosts: servers
  roles:
    - { role: prometheus, action: "postgres_exporter" }
```

## License

MIT

## Author Information

SmartLogic. https://smartlogic.io
