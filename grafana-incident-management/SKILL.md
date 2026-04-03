---
name: grafana-incident-management
description: Create, list, and manage incidents using Grafana IRM (Incident Response Management), including OnCall schedules and shift information.
---

# Grafana Incident Management Skill

This skill helps you work with Grafana IRM (Incident Response Management) — creating and tracking incidents, managing on-call schedules, and coordinating incident response.

## Available Operations

### Incidents

#### List Incidents
Use `list_incidents` to retrieve all active or recent incidents.

```
list_incidents()
```

#### Get a Specific Incident
Use `get_incident` to retrieve details about a specific incident.

```
get_incident(incident_id="incident-id")
```

#### Create an Incident
Use `create_incident` to open a new incident.

```
create_incident(
  title="Database connection failures in production",
  severity="critical",
  labels={"service": "database", "environment": "production"}
)
```

#### Add Activity to an Incident
Use `add_activity_to_incident` to add a timeline entry or update to an incident.

```
add_activity_to_incident(
  incident_id="incident-id",
  activity_kind="userNote",
  body="Identified root cause: memory leak in connection pool"
)
```

### OnCall

#### List OnCall Users
Use `list_oncall_users` to see all users in the OnCall system.

```
list_oncall_users()
```

#### List OnCall Teams
Use `list_oncall_teams` to see all OnCall teams.

```
list_oncall_teams()
```

#### List OnCall Schedules
Use `list_oncall_schedules` to see all on-call rotation schedules.

```
list_oncall_schedules()
```

#### Get Current OnCall Users
Use `get_current_oncall_users` to find who is currently on call for a specific schedule.

```
get_current_oncall_users(schedule_id="schedule-id")
```

#### Get a Specific Shift
Use `get_oncall_shift` to get details about a specific on-call shift.

```
get_oncall_shift(shift_id="shift-id")
```

### Sift Investigations (AI-powered)

#### List Sift Investigations
Use `list_sift_investigations` to see AI-powered incident investigations.

```
list_sift_investigations()
```

#### Get a Sift Investigation
Use `get_sift_investigation` to get details about a specific Sift investigation.

```
get_sift_investigation(investigation_id="investigation-id")
```

#### Get Sift Analysis
Use `get_sift_analysis` to retrieve the analysis results from a Sift investigation.

```
get_sift_analysis(investigation_id="investigation-id")
```

## Incident Severity Levels

| Severity | Description |
|----------|-------------|
| `critical` | Complete service outage, revenue impact |
| `high` | Major functionality impaired |
| `medium` | Partial functionality impaired |
| `low` | Minor issue, workaround available |

## Common Workflows

### Respond to a Production Alert
1. Call `list_incidents()` to check if an incident is already open
2. If not, use `create_incident` with appropriate severity and title
3. Add initial investigation notes with `add_activity_to_incident`
4. Use `get_current_oncall_users` to identify the right responders
5. Continue adding activity entries as the investigation progresses

### Incident Handoff
1. Use `get_incident` to get the current state
2. Review the activity timeline to understand what's been done
3. Add a handoff note with `add_activity_to_incident`
4. Use `get_current_oncall_users` to identify the next on-call responder

### Post-Incident Review
1. Use `get_incident` to retrieve the full incident details
2. Review the complete activity timeline
3. Check the Sift investigation (if available) with `get_sift_investigation`
4. Use the information to document the post-mortem

### Find Who Is On Call
1. Call `list_oncall_schedules()` to get all schedules
2. Identify the relevant schedule by name/team
3. Call `get_current_oncall_users(schedule_id=...)` for the active responders
4. Or use `list_oncall_teams()` to find team-specific schedules

## Best Practices
- Create incidents promptly when production services are impacted
- Use clear, descriptive incident titles that explain the impact
- Set the correct severity to drive appropriate escalation
- Add timeline entries frequently to document the investigation
- Link related alerts and dashboards in the incident
- Include the affected service and environment in labels
- Close incidents promptly when resolved, with a resolution summary
