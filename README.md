# kagent-grafana-skills

Git-based [kagent](https://github.com/kagent-dev/kagent) skills for Grafana observability — dashboards, alerting, Prometheus querying, Loki log querying, and incident management.

## Available Skills

| Skill | Description |
|-------|-------------|
| [`grafana-dashboard-management`](./grafana-dashboard-management/SKILL.md) | Create, search, update, delete, and manage Grafana dashboards including panels, versions, and permissions |
| [`grafana-datasource-management`](./grafana-datasource-management/SKILL.md) | List, retrieve, and inspect Grafana data sources including Prometheus, Loki, and other configured backends |
| [`grafana-alerting`](./grafana-alerting/SKILL.md) | List, inspect, and understand Grafana alert rules and contact points for monitoring and notification configuration |
| [`grafana-incident-management`](./grafana-incident-management/SKILL.md) | Create, list, and manage incidents using Grafana IRM, including OnCall schedules and shift information |
| [`prometheus-querying`](./prometheus-querying/SKILL.md) | Query Prometheus metrics using PromQL, list available metrics and labels, and interpret time series data |
| [`loki-log-querying`](./loki-log-querying/SKILL.md) | Query Loki for application and infrastructure logs using LogQL, filter and search logs, and retrieve statistics |
| [`grafana-apm-skill`](./grafana-apm-skill/SKILL.md) | Monitor application performance using Grafana APM tools — query distributed traces with Tempo, analyze profiling data with Pyroscope, and compute RED metrics for services |

## Using These Skills with kagent

Reference this repository in your kagent `Agent` resource using `gitRefs`:

```yaml
apiVersion: kagent.dev/v1alpha2
kind: Agent
metadata:
  name: grafana-agent
  namespace: kagent
spec:
  type: Declarative
  description: Grafana observability agent with dashboard, alerting, and incident management skills
  skills:
    gitRefs:
      - url: https://github.com/bhagyesh-raut-incontact/kagent-grafana-skills
        ref: main
  declarative:
    modelConfig: default-model-config
    systemMessage: |
      You are a Grafana observability agent. You have access to skills for managing
      Grafana dashboards, querying Prometheus metrics, searching Loki logs, managing
      alerts, and handling incidents.

      Use the skills tool to load detailed instructions for each capability.
    tools:
      - type: McpServer
        mcpServer:
          name: kagent-grafana-mcp
          kind: RemoteMCPServer
          apiGroup: kagent.dev
```

### Pinning to a Specific Version

To pin to a specific commit or tag for reproducibility:

```yaml
skills:
  gitRefs:
    - url: https://github.com/bhagyesh-raut-incontact/kagent-grafana-skills
      ref: v1.0.0   # tag, branch, or commit SHA
```

### Using a Subdirectory

If you only want a subset of skills, use the `path` field to point to a subdirectory:

```yaml
skills:
  gitRefs:
    - url: https://github.com/bhagyesh-raut-incontact/kagent-grafana-skills
      ref: main
      path: grafana-dashboard-management
      name: grafana-dashboards
```

## Skill File Format

Each skill is a directory containing a `SKILL.md` file with YAML frontmatter:

```
grafana-dashboard-management/
└── SKILL.md        # Metadata (YAML frontmatter) + instructions
```

The `SKILL.md` format:

```markdown
---
name: skill-name
description: Short description of what the skill does
---

# Skill Title

Full instructions for the agent...
```

The kagent `SkillsTool` automatically discovers and loads these skills from the cloned repository directory.
