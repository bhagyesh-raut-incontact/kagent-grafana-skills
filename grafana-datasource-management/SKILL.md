---
name: grafana-datasource-management
description: List, retrieve, and inspect Grafana data sources including Prometheus, Loki, and other configured backends.
---

# Grafana Datasource Management Skill

This skill helps you manage and inspect Grafana data sources — listing available sources, retrieving their details, and understanding their configuration for use in dashboards and queries.

## Available Operations

### List All Datasources
Use `list_datasources` to get all configured data sources in Grafana.

```
list_datasources()
```

Returns a list of data sources with their names, types, UIDs, and URLs.

### Get Datasource by Name
Use `get_datasource_by_name` to retrieve a specific data source by its name.

```
get_datasource_by_name(name="Prometheus")
```

### Get Datasource by UID
Use `get_datasource_by_uid` to retrieve a specific data source by its UID.

```
get_datasource_by_uid(uid="abc123")
```

## Common Datasource Types

| Type | Name | Purpose |
|------|------|---------|
| `prometheus` | Prometheus | Time series metrics |
| `loki` | Loki | Log aggregation |
| `elasticsearch` | Elasticsearch | Log/metric search |
| `influxdb` | InfluxDB | Time series DB |
| `graphite` | Graphite | Time series metrics |
| `cloudwatch` | CloudWatch | AWS metrics |
| `azuremonitor` | Azure Monitor | Azure metrics |
| `tempo` | Tempo | Distributed tracing |

## Common Workflows

### Find the Prometheus Datasource UID
Before creating dashboard panels, you need the Prometheus datasource UID for the `datasource` field:

1. Call `list_datasources()`
2. Find the entry with `type: prometheus`
3. Copy the `uid` field for use in panel configurations

### Verify Datasource Availability
Before building a dashboard that relies on a specific datasource:
1. Call `list_datasources()` to confirm the datasource exists
2. Check the datasource `type` matches what you need
3. Note the `uid` for panel target configurations

### Use Datasource in Dashboard Panels
When creating panels, reference the datasource by UID:
```json
{
  "datasource": {
    "type": "prometheus",
    "uid": "the-uid-from-list-datasources"
  }
}
```

For dashboard-wide variable (best practice):
```json
{
  "templating": {
    "list": [
      {
        "name": "datasource",
        "type": "datasource",
        "query": "prometheus",
        "label": "Data Source"
      }
    ]
  }
}
```
Then reference it as `"uid": "${datasource}"` in each panel.

## Best Practices
- Always retrieve and use the actual datasource UID rather than hardcoding names
- Use dashboard variables for the datasource to make dashboards portable
- Confirm datasource availability before creating dashboards that depend on it
- When multiple Prometheus instances exist, identify the correct one by name or URL
