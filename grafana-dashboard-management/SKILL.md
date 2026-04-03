---
name: grafana-dashboard-management
description: Create, search, update, delete, and manage Grafana dashboards including panels, versions, and permissions.
---

# Grafana Dashboard Management Skill

This skill helps you manage Grafana dashboards — creating new dashboards from user requirements, finding existing ones, updating panels, managing versions, and configuring permissions.

## Available Operations

### Search Dashboards
Use `search_dashboards` to find dashboards by title, tag, or folder.

```
search_dashboards(query="kubernetes", type="dash-db")
```

### Get a Dashboard
Use `get_dashboard_by_uid` or `get_dashboard_by_title` to retrieve a specific dashboard.

```
get_dashboard_by_uid(uid="abc123")
```

### Create or Update a Dashboard
Use `update_dashboard` to create a new dashboard or update an existing one. Pass a complete Grafana dashboard JSON model.

**Important dashboard JSON structure:**
```json
{
  "dashboard": {
    "id": null,
    "uid": null,
    "title": "My Dashboard",
    "tags": ["kubernetes", "monitoring"],
    "timezone": "browser",
    "panels": [],
    "refresh": "30s",
    "time": { "from": "now-1h", "to": "now" },
    "schemaVersion": 36
  },
  "folderId": 0,
  "overwrite": true,
  "message": "Created via kagent"
}
```

### Panel Types
Common panel types for Grafana:
- `timeseries` — Time series line/bar charts (most common for metrics)
- `gauge` — Single stat gauges
- `stat` — Single statistics
- `table` — Tabular data
- `piechart` — Pie/donut charts
- `barchart` — Bar charts
- `heatmap` — Heatmaps
- `logs` — Log panels (for Loki)

### Panel JSON Structure
```json
{
  "id": 1,
  "title": "CPU Usage",
  "type": "timeseries",
  "datasource": { "type": "prometheus", "uid": "${datasource}" },
  "targets": [
    {
      "expr": "rate(container_cpu_usage_seconds_total[5m])",
      "legendFormat": "{{pod}}",
      "refId": "A"
    }
  ],
  "gridPos": { "x": 0, "y": 0, "w": 12, "h": 8 },
  "fieldConfig": {
    "defaults": {
      "unit": "percentunit",
      "min": 0,
      "max": 1
    }
  }
}
```

### Manage Dashboard Versions
```
get_dashboard_versions(uid="abc123")
get_dashboard_version(uid="abc123", version=3)
restore_dashboard_version(uid="abc123", version=2)
```

### Dashboard Permissions
```
get_dashboard_permissions(uid="abc123")
update_dashboard_permissions(uid="abc123", permissions=[...])
```

### Compare Dashboard Versions
```
calculate_dashboard_diff(base={"dashboardId": 1, "version": 1}, new={"dashboardId": 1, "version": 2})
```

## Common Workflows

### Create a New Dashboard for Kubernetes Metrics
1. Use `list_datasources` to find the Prometheus datasource UID
2. Design panels for the required metrics (CPU, memory, network, etc.)
3. Build the complete dashboard JSON model
4. Call `update_dashboard` with `"id": null` to create new

### Update an Existing Dashboard
1. Use `get_dashboard_by_uid` to retrieve the current version
2. Modify the panels or settings
3. Call `update_dashboard` with the existing UID to update
4. Set `"overwrite": true` to replace the current version

### Add a Panel to an Existing Dashboard
1. Fetch the dashboard with `get_dashboard_by_uid`
2. Append a new panel object to the `panels` array
3. Assign a unique `id` and appropriate `gridPos`
4. Save with `update_dashboard`

## Best Practices
- Always set meaningful titles and tags for discoverability
- Use dashboard variables (`$datasource`, `$namespace`) for reusability
- Set appropriate time ranges and refresh intervals
- Use template variables for filtering by namespace, pod, or service
- Version control dashboard changes with descriptive commit messages
- Test PromQL queries in the Explore view before adding to dashboards
