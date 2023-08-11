# Prometheus Exporters

Set up common prometheus exporter configurations

## Install to requirements.yml

```
- src: git+git@github.com:smartlogic/ansible-role-prometheus-exporters
  name: prometheus-exporters
  version: 1.1.0
```

## Requirements

### For postgres_exporter

Set up permission for the postgres_exporter user (by default) to have access to the necessary stats, update the password to something more secure

```sql
CREATE OR REPLACE FUNCTION __tmp_create_user() returns void as $$
BEGIN
  IF NOT EXISTS (
          SELECT                       -- SELECT list can stay empty for this
          FROM   pg_catalog.pg_user
          WHERE  usename = 'postgres_exporter') THEN
    CREATE USER postgres_exporter;
  END IF;
END;
$$ language plpgsql;

SELECT __tmp_create_user();
DROP FUNCTION __tmp_create_user();

ALTER USER postgres_exporter WITH PASSWORD '<set-a-password>';
ALTER USER postgres_exporter SET SEARCH_PATH TO postgres_exporter,pg_catalog;

-- If deploying as non-superuser (for example in AWS RDS), uncomment the GRANT
-- line below and replace <MASTER_USER> with your root user.
-- GRANT postgres_exporter TO <MASTER_USER>;

GRANT CONNECT ON DATABASE postgres TO postgres_exporter;
```

### Firewall Ports

These ports should be opened to the prometheus server based on the enabled exporters:

| Exporter      | Port  |
| ------------- | ----- |
| node          | 9100  |
| statsd        | 9102  |
| elasticsearch | 9114  |
| blackbox      | 9115  |
| redis         | 9121  |
| postgres      | 9187  |

## Role Variables

- `node_exporter_version` - Which version of node exporter to download
- `node_exporter_checksum` - The checksum for the version of node exporter
- `node_exporter_env_file` - A file, default to empty, can be in your playbook directory, that is the env file for the exporter, used to set `NODE_EXPORTER_ARGS` for command line args or other env vars for node exporter
- `redis_exporter_version` - Which version of redis_exporter to download
- `redis_exporter_checksum` - The checksum for the version of redis_exporter
- `redis_addr` - The address of the redis instance to monitor
  - Default: `redis://localhost:6379`
- `postgres_exporter_version` - Which version of postgres_exporter to download
- `postgres_exporter_checksum` - The checksum for the version of postgres_exporter
- `postgres_exporter_connection_host` - The host to connect to
  - Default: `/var/run/postgresql/` to use the default postgres socket
- `postgres_exporter_connection_ssl_mode` - If to use ssl when connecting
  - Default: `disable` - because the default is socket connection
- `postgres_exporter_connection_user` -
  - Default: `prometheus` - the connecting user (uses identity be default)
- `postgres_exporter_data_source_uri` - use URI style with username and password ommited, used in place of the previous options
- `postgres_exporter_data_source_user` - use with URI style to specify the user
- `postgres_exporter_data_source_pass` - use with URI style to specify the password
- `blackbox_exporter_version` - Which version of blackbox_exporter to download
- `blackbox_exporter_checksum` - The checksum for the version of blackbox_exporter
- `statsd_exporter_version` - Which version of statsd_exporter to download
- `statsd_exporter_checksum` - The checksum for the version of statsd_exporter
- `statsd_mapping_file` - The file to use for statsd mapping to prometheus metrics
  - Default: an empty mappings file
- `elasticsearch_exporter_version` - Which version of elasticsearch_exporter to download
- `elasticsearch_exporter_checksum` - The checksum for the version of elasticsearch_exporter
- `elasticsearch_exporter_listen_addr` - What interface and port to listen on, defaults to `:9114`
- `elasticsearch_exporter_uri` - The URI of the ES node to connect to, should include `username:password@` if necessary, default is `http://localhost:9200`

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
