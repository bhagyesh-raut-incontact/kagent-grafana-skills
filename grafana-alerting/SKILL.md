---
name: grafana-alerting
description: List, inspect, and understand Grafana alert rules and contact points for monitoring and notification configuration.
---

# Grafana Alerting Skill

This skill helps you work with Grafana's alerting system — listing alert rules, inspecting their configuration, checking alert states, and reviewing notification contact points.

## Available Operations

### List Alert Rules
Use `list_alert_rules` to retrieve all configured alert rules.

```
list_alert_rules()
```

Returns alert rules with their names, states, conditions, and evaluation intervals.

### Get a Specific Alert Rule
Use `get_alert_rule_by_uid` to retrieve the details of a specific alert rule.

```
get_alert_rule_by_uid(uid="alert-rule-uid")
```

### List Contact Points
Use `list_contact_points` to see all configured notification channels (email, Slack, PagerDuty, etc.).

```
list_contact_points()
```

## Alert Rule Structure

A Grafana alert rule contains:
- **Name** — Human-readable name for the alert
- **Group** — Alert group for organizing related alerts
- **Folder** — Namespace/folder for the rule
- **Condition** — Which data query exceeds a threshold
- **Evaluation interval** — How often the rule is evaluated
- **For** — How long the condition must be true before firing
- **Labels** — Key-value pairs for routing/grouping
- **Annotations** — Additional context (description, runbook URL)

## Alert States

| State | Meaning |
|-------|---------|
| `OK` / `Normal` | Condition not met, alert not firing |
| `Pending` | Condition met but within the "for" duration |
| `Alerting` / `Firing` | Alert is active and notifications sent |
| `NoData` | Query returned no data |
| `Error` | Alert evaluation failed |

## Common Alert Patterns

### High CPU Usage Alert
```
Query: avg(rate(container_cpu_usage_seconds_total[5m])) by (pod) > 0.8
For: 5m
Labels: severity=warning
```

### Pod Restart Alert
```
Query: increase(kube_pod_container_status_restarts_total[1h]) > 5
For: 0m
Labels: severity=critical
```

### Memory Usage Alert
```
Query: container_memory_usage_bytes / container_spec_memory_limit_bytes > 0.9
For: 10m
Labels: severity=warning
```

### Service Down Alert
```
Query: up{job="my-service"} == 0
For: 1m
Labels: severity=critical
```

## Common Workflows

### Audit Current Alert Rules
1. Call `list_alert_rules()` to get all configured rules
2. Review each rule's condition, thresholds, and evaluation settings
3. Use `get_alert_rule_by_uid` for detailed inspection of specific rules

### Check Notification Configuration
1. Call `list_contact_points()` to see available notification channels
2. Verify the appropriate channels exist for alerting (email, Slack, PagerDuty)
3. Cross-reference with alert rule routing policies

### Investigate Firing Alerts
1. Call `list_alert_rules()` and filter for rules in `Alerting` state
2. Use `get_alert_rule_by_uid` to see the exact condition that fired
3. Cross-reference with Prometheus metrics using the `prometheus-querying` skill

## Best Practices
- Use meaningful alert names that describe the problem, not the symptom
- Set appropriate "for" durations to avoid flapping (transient alerts)
- Include runbook URLs in annotations so responders know what to do
- Group related alerts together for cleaner routing
- Use labels (severity, team, service) for fine-grained routing to contact points
- Test alert conditions with PromQL in the Explore view before creating rules
- Set NoData and Error behaviors explicitly to avoid silent failures
