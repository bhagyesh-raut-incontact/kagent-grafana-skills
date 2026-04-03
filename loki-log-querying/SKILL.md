---
name: loki-log-querying
description: Query Loki for application and infrastructure logs using LogQL, filter and search logs, and retrieve log statistics.
---

# Loki Log Querying Skill

This skill helps you query Grafana Loki for logs using LogQL — searching application logs, filtering by labels, and retrieving log statistics for troubleshooting and monitoring.

## Available Operations

### Query Logs
Use `query_loki_logs` to retrieve log lines matching a LogQL expression.

```
query_loki_logs(query='{namespace="production", app="my-service"} |= "error"')
```

Optional parameters:
- `start` — Start time (e.g., `"now-1h"`, `"2024-01-01T00:00:00Z"`)
- `end` — End time (e.g., `"now"`)
- `limit` — Maximum number of log lines to return

### Query Log Statistics
Use `query_loki_stats` to get statistical aggregations over log streams.

```
query_loki_stats(query='sum(rate({namespace="production"}[5m])) by (app)')
```

### List Label Names
Use `list_loki_label_names` to discover available log label keys.

```
list_loki_label_names()
```

### List Label Values
Use `list_loki_label_values` to see all values for a specific label.

```
list_loki_label_values(label_name="namespace")
list_loki_label_values(label_name="app")
```

## LogQL Quick Reference

### Log Stream Selectors

Log streams are selected with label matchers in `{}`:

```
{namespace="production"}                     # Exact match
{app="my-service", environment="prod"}       # Multiple labels (AND)
{app=~"my-service|other-service"}            # Regex match
{namespace!="kube-system"}                   # Exclude label value
{container=~".+"}                            # Label must exist
```

### Log Pipeline Filters

Chain filters after the stream selector using `|`:

```
{app="my-service"} |= "error"                # Contains string
{app="my-service"} != "debug"                # Does not contain string
{app="my-service"} |~ "error|warning"        # Regex match
{app="my-service"} !~ "health.*check"        # Regex not match
```

### Parsing Log Lines

**JSON logs:**
```
{app="my-service"} | json                    # Parse as JSON
{app="my-service"} | json level, message     # Parse specific fields
{app="my-service"} | json | level="error"    # Parse then filter
```

**Logfmt (key=value format):**
```
{app="my-service"} | logfmt                  # Parse logfmt
{app="my-service"} | logfmt | level="error"  # Parse then filter
```

**Pattern:**
```
{app="nginx"} | pattern '<ip> - <user> [<date>] "<method> <path> <proto>" <status> <size>'
```

**Regular Expression:**
```
{app="my-service"} | regexp `(?P<level>\w+): (?P<message>.+)`
```

### Metric Queries (for statistics)

**Log rate:**
```
rate({namespace="production"}[5m])           # Logs per second
sum(rate({namespace="production"}[5m])) by (app)  # By app
```

**Count over time:**
```
count_over_time({app="my-service"} |= "error" [1h])  # Error count per hour
```

**Bytes rate:**
```
bytes_rate({app="my-service"}[5m])           # Bytes per second
```

**Quantile over time:**
```
quantile_over_time(0.99, {app="my-service"} | json | unwrap response_time [5m]) by (endpoint)
```

## Common Query Patterns

### Find Error Logs
```
{namespace="production"} |= "error" | json | level="error"
```

### Kubernetes Pod Logs
```
{namespace="production", pod="my-pod-abc123"}
```

### Logs for a Deployment
```
{namespace="production", app="my-service"}
```

### Recent Errors with Context
```
{app="my-service"} |= "error" | json
```

### HTTP 5xx Errors (JSON logs)
```
{app="api-gateway"} | json | status >= 500
```

### Exception Stack Traces
```
{app="my-service"} |~ "Exception|Error|panic"
```

### Log Volume by App
```
sum(rate({namespace="production"}[5m])) by (app)
```

### Error Rate by Service
```
sum(rate({namespace="production"} |= "error" [5m])) by (app)
  /
sum(rate({namespace="production"}[5m])) by (app)
```

### Top Log-Producing Pods
```
topk(10, sum(rate({namespace="production"}[5m])) by (pod))
```

## Common Workflows

### Troubleshoot a Failed Service
1. Call `list_loki_label_values(label_name="app")` to find the service label
2. Query recent logs: `{app="my-service", namespace="production"}`
3. Filter for errors: `{app="my-service"} |= "error"`
4. If JSON logs, parse and filter: `{app="my-service"} | json | level="error"`
5. Look for patterns or specific error messages

### Investigate a Kubernetes Pod
1. Get the pod name from the user or Kubernetes
2. Query: `{namespace="production", pod="pod-name"}`
3. Filter for errors or warnings
4. Check timestamps around the incident time

### Analyze Log Volume Spike
1. Use `query_loki_stats` with a rate query:
   `sum(rate({namespace="production"}[5m])) by (app)`
2. Identify which apps/pods are generating excessive logs
3. Drill down with filtered queries on those specific services

### Find Error Patterns
1. Use `find_error_pattern_logs` for automated pattern detection (if available)
2. Or query: `{app="my-service"} |= "error" | json`
3. Look for repeating error messages
4. Aggregate by error type or message

### Check Application Startup Logs
```
{app="my-service"} |~ "started|initialized|ready"
```

## Best Practices
- Always start with a stream selector to limit the query scope
- Add time bounds to avoid scanning all historical data
- Use `| json` or `| logfmt` to parse structured logs for field-level filtering
- Use `rate()` in metric queries rather than raw counts for alerting
- Limit results with `limit` parameter when exploring logs interactively
- Use label selectors that match your log format (check labels with `list_loki_label_names`)
- Combine `query_loki_logs` (for log lines) and `query_loki_stats` (for aggregations)
