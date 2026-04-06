---
name: grafana-apm-skill
description: Monitor application performance using Grafana APM tools — query distributed traces with Tempo, analyze profiling data with Pyroscope, and compute RED metrics (Rate, Errors, Duration) for services.
---

# Grafana APM Skill

This skill helps you monitor application performance using Grafana's APM stack — querying distributed traces from Tempo, profiling data from Pyroscope, and computing RED (Rate, Errors, Duration) metrics from Prometheus.

## Available Operations

### Trace Querying (Grafana Tempo)

#### Search Traces
Use `search_tempo_traces` to find traces matching a filter.

```
search_tempo_traces(service_name="checkout", duration_min="500ms", limit=20)
search_tempo_traces(tags={"http.status_code": "500"})
```

#### Get a Specific Trace
Use `get_tempo_trace_by_id` to retrieve all spans for a trace.

```
get_tempo_trace_by_id(trace_id="3c0e1b4f8a2d9e6c")
```

#### Search Tags
Use `search_tempo_tags` to list available tag names for filtering.

```
search_tempo_tags()
```

#### Search Tag Values
Use `search_tempo_tag_values` to see all values for a specific tag.

```
search_tempo_tag_values(tag="service.name")
search_tempo_tag_values(tag="http.route")
```

### Profiling (Grafana Pyroscope)

#### Query Profile Data
Use `query_pyroscope_profile` to retrieve CPU, memory, or goroutine profiling data for a service.

```
query_pyroscope_profile(service_name="api-server", profile_type="cpu", from="now-1h", to="now")
query_pyroscope_profile(service_name="worker", profile_type="memory:inuse_space:bytes:space:bytes")
```

#### List Services
Use `list_pyroscope_services` to see all profiled services.

```
list_pyroscope_services()
```

#### List Profile Types
Use `list_pyroscope_profile_types` to see what profiling data is available for a service.

```
list_pyroscope_profile_types(service_name="api-server")
```

### RED Metrics (via Prometheus)

Use `query_prometheus` to compute RED metrics for any instrumented service.

**Rate — requests per second:**
```
sum(rate(http_requests_total{service="checkout"}[5m])) by (service)
```

**Errors — error rate:**
```
sum(rate(http_requests_total{service="checkout", status=~"5.."}[5m])) by (service)
  / sum(rate(http_requests_total{service="checkout"}[5m])) by (service)
```

**Duration — 99th percentile latency:**
```
histogram_quantile(0.99,
  sum(rate(http_request_duration_seconds_bucket{service="checkout"}[5m])) by (service, le)
)
```

## TraceQL Quick Reference (Tempo)

TraceQL is Tempo's query language for filtering and aggregating trace data.

### Basic Span Filters

```
{ .service.name = "frontend" }
{ .http.status_code >= 500 }
{ .http.method = "POST" && duration > 1s }
{ status = error }
```

### Structural Operators

```
# Child span must match
{ .service.name = "gateway" } >> { .service.name = "payments" }

# Ancestor span must match
{ status = error } && ancestor({ .service.name = "api" })
```

### Aggregation

```
# Count spans per service
count_over_time({ .service.name =~ ".+" }[5m]) by (.service.name)

# P99 duration per service
quantile_over_time(0.99, { status = ok }[5m], duration) by (.service.name)
```

## Common APM Workflows

### Investigate a Slow Request
1. Use `search_tempo_traces(service_name="my-service", duration_min="2s")` to find slow traces.
2. Call `get_tempo_trace_by_id(trace_id="...")` to inspect the full span tree.
3. Identify which span contributes the most latency.
4. Query `query_prometheus` with RED metrics for that service to confirm the pattern at scale.

### Triage a Spike in Errors
1. Query Prometheus error rate: `sum(rate(http_requests_total{status=~"5.."}[5m])) by (service)`.
2. Use `search_tempo_traces(tags={"http.status_code": "500"})` to find failing traces.
3. Inspect spans in the failing traces with `get_tempo_trace_by_id` to find the root cause service.

### Profile a CPU-Heavy Service
1. Call `list_pyroscope_services()` to confirm the service is being profiled.
2. Call `list_pyroscope_profile_types(service_name="worker")` to list available profiles.
3. Call `query_pyroscope_profile(service_name="worker", profile_type="cpu", from="now-30m", to="now")` to retrieve the flame graph data.
4. Identify the hottest call stacks and cross-reference with recent deployments.

### Build a Service Map
1. Query Prometheus for all services: `group(rate(http_requests_total[5m])) by (service)`.
2. Use `search_tempo_tag_values(tag="service.name")` to list services in Tempo.
3. For each service pair, query Tempo with a child-span filter to establish call relationships:
   ```
   { .service.name = "frontend" } >> { .service.name = "backend" }
   ```

## RED Metric Reference

| Signal | PromQL Pattern | Description |
|--------|---------------|-------------|
| Rate | `rate(requests_total[5m])` | Requests per second |
| Error rate | `rate(errors_total[5m]) / rate(requests_total[5m])` | Fraction of failing requests |
| P50 latency | `histogram_quantile(0.50, rate(duration_bucket[5m]))` | Median response time |
| P95 latency | `histogram_quantile(0.95, rate(duration_bucket[5m]))` | 95th percentile response time |
| P99 latency | `histogram_quantile(0.99, rate(duration_bucket[5m]))` | 99th percentile response time |

## Best Practices

- Always correlate traces with metrics: find the anomaly in metrics first, then drill into traces for root cause.
- Use `duration_min` filters in Tempo to surface only slow traces and reduce noise.
- Filter traces by `status = error` to quickly find all failing requests without scanning every trace.
- Pair Pyroscope profiling with Tempo tracing to identify both *which* service is slow and *why* (CPU, memory, I/O).
- Use RED metrics as your default SLI baseline — they cover the most common failure modes for any HTTP/gRPC service.
- Set Tempo query time ranges consistent with your alert windows (e.g., last 5 minutes for active incidents, last 1 hour for trend analysis).
