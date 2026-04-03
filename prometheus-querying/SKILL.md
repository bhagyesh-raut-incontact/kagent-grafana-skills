---
name: prometheus-querying
description: Query Prometheus metrics using PromQL, list available metrics and labels, and interpret time series data for monitoring and alerting.
---

# Prometheus Querying Skill

This skill helps you query Prometheus for metrics using PromQL — executing queries, listing available metrics and labels, and interpreting the results for monitoring and troubleshooting.

## Available Operations

### Execute a Query
Use `query_prometheus` to run a PromQL query against Prometheus.

```
query_prometheus(expr="sum(rate(http_requests_total[5m])) by (service)")
```

### List Metric Names
Use `list_prometheus_metric_names` to discover all available metrics.

```
list_prometheus_metric_names()
```

Use the optional `match` parameter to filter:
```
list_prometheus_metric_names(match="container_cpu")
```

### Get Metric Metadata
Use `list_prometheus_metric_metadata` to see metric types and descriptions.

```
list_prometheus_metric_metadata(metric="http_requests_total")
```

### List Label Names
Use `list_prometheus_label_names` to see all available label keys.

```
list_prometheus_label_names()
```

### List Label Values
Use `list_prometheus_label_values` to see all values for a specific label.

```
list_prometheus_label_values(label_name="namespace")
list_prometheus_label_values(label_name="pod", match="app=my-service")
```

## PromQL Quick Reference

### Metric Types

| Type | Description | Example |
|------|-------------|---------|
| Counter | Always increasing | `http_requests_total` |
| Gauge | Can go up or down | `container_memory_usage_bytes` |
| Histogram | Distribution of values | `http_request_duration_seconds` |
| Summary | Quantiles over time | `go_gc_duration_seconds` |

### Essential PromQL Functions

| Function | Use Case | Example |
|----------|----------|---------|
| `rate()` | Per-second rate for counters | `rate(http_requests_total[5m])` |
| `irate()` | Instant rate (spiky data) | `irate(http_requests_total[2m])` |
| `increase()` | Total increase over range | `increase(errors_total[1h])` |
| `sum()` | Aggregate by summing | `sum(rate(requests[5m])) by (service)` |
| `avg()` | Aggregate by averaging | `avg(cpu_usage) by (node)` |
| `max()` | Maximum value | `max(memory_bytes) by (pod)` |
| `min()` | Minimum value | `min(disk_free_bytes) by (node)` |
| `count()` | Count series | `count(up) by (job)` |
| `histogram_quantile()` | Percentile from histogram | `histogram_quantile(0.99, rate(duration_bucket[5m]))` |
| `abs()` | Absolute value | `abs(delta(temp[1h]))` |
| `predict_linear()` | Linear prediction | `predict_linear(disk_bytes[1h], 4*3600)` |

### Label Matchers

```
metric{label="exact"}          # Exact match
metric{label!="value"}         # Not equal
metric{label=~"pattern.*"}     # Regex match
metric{label!~"pattern.*"}     # Regex not match
```

## Common Query Patterns

### Kubernetes Resource Monitoring

**CPU Usage by Pod:**
```
sum(rate(container_cpu_usage_seconds_total{container!=""}[5m])) by (pod, namespace)
```

**Memory Usage by Pod:**
```
container_memory_working_set_bytes{container!=""} / 1024 / 1024
```

**CPU Usage as % of Limit:**
```
sum(rate(container_cpu_usage_seconds_total[5m])) by (pod)
  / sum(kube_pod_container_resource_limits{resource="cpu"}) by (pod)
```

**Memory as % of Limit:**
```
container_memory_working_set_bytes
  / kube_pod_container_resource_limits{resource="memory"}
```

**Pod Restarts (last hour):**
```
increase(kube_pod_container_status_restarts_total[1h]) > 0
```

**Pods Not Ready:**
```
kube_pod_status_ready{condition="true"} == 0
```

### HTTP / Service Metrics

**Request Rate:**
```
sum(rate(http_requests_total[5m])) by (service, method, status)
```

**Error Rate:**
```
sum(rate(http_requests_total{status=~"5.."}[5m])) by (service)
  / sum(rate(http_requests_total[5m])) by (service)
```

**99th Percentile Latency:**
```
histogram_quantile(0.99,
  sum(rate(http_request_duration_seconds_bucket[5m])) by (service, le)
)
```

**Average Latency:**
```
sum(rate(http_request_duration_seconds_sum[5m])) by (service)
  / sum(rate(http_request_duration_seconds_count[5m])) by (service)
```

### Infrastructure Metrics

**Node CPU Usage:**
```
1 - avg(rate(node_cpu_seconds_total{mode="idle"}[5m])) by (instance)
```

**Node Memory Available:**
```
node_memory_MemAvailable_bytes / node_memory_MemTotal_bytes
```

**Disk Usage:**
```
1 - (node_filesystem_avail_bytes / node_filesystem_size_bytes)
```

**Network Traffic (in/out):**
```
sum(rate(node_network_receive_bytes_total[5m])) by (instance)
sum(rate(node_network_transmit_bytes_total[5m])) by (instance)
```

## Common Workflows

### Investigate High CPU Usage
1. Call `list_prometheus_label_values(label_name="namespace")` to find namespaces
2. Query CPU by namespace: `sum(rate(container_cpu_usage_seconds_total[5m])) by (pod, namespace)`
3. Identify the top consumers and drill down with more specific queries

### Calculate Error Rate for a Service
1. Query total requests: `sum(rate(http_requests_total{service="my-svc"}[5m]))`
2. Query error requests: `sum(rate(http_requests_total{service="my-svc", status=~"5.."}[5m]))`
3. Calculate ratio for the error rate percentage

### Find Metrics for a Service
1. Use `list_prometheus_metric_names(match="my_service")` to find relevant metrics
2. Use `list_prometheus_metric_metadata(metric="metric_name")` to understand each metric
3. Use `list_prometheus_label_values(label_name="job")` to find the job label for the service

## Best Practices
- Always use `rate()` with counters, not raw counter values
- Choose an appropriate time range for `rate()` — at least 4x the scrape interval
- Use `by` clause to aggregate by relevant dimensions
- Filter with label selectors to reduce cardinality
- Use `histogram_quantile()` for latency percentiles instead of averages
- Prefer `container!=""` filter to exclude pause containers in Kubernetes queries
