# Jira JSM On-Call Configuration Schemas

## Introduction

Jira Service Management (JSM) provides a powerful framework for managing incidents and on-call schedules through its On-Call Management feature. To effectively leverage this capability, understanding and configuring the appropriate schemas is crucial. This document offers an exhaustive guide to the configuration schemas used within Jira JSM On-Call. Covering every configuration file, its fields, default values, and best practices, this guide is intended for system administrators, developers, and technical users who intend to tailor JSM On-Call to their organizational needs.

## Configuration Overview

Jira JSM On-Call uses a series of configuration files to manage various aspects of its operation, including schedules, escalation policies, notifications, and integration settings. These configurations are typically maintained in JSON or YAML formats, allowing for structured and easily readable setups.

The primary configuration components include:

- On-Call Schedule Configuration
- Escalation Policy Configuration
- Notification Rules Configuration
- Integration Configuration

In the sections that follow, each of these configuration types will be explored in detail.

---

## On-Call Schedule Configuration

### on-call-schedule-config.yaml

The `on-call-schedule-config.yaml` file defines the schedules for various teams within Jira JSM. It includes details such as rotation names, time zones, rotation policies, and participants.

#### Fields

- **schedule_id** (string, required): A unique identifier for the schedule.
- **name** (string, required): The name of the on-call schedule.
- **timezone** (string, default: "UTC"): The time zone in which the schedule operates.
- **rotations** (array, required): A list of rotation configurations.

##### Rotation Configuration

Each rotation within the on-call schedule can be configured with the following fields:

- **name** (string, required): The name of the rotation.
- **start_time** (string, required): ISO 8601 formatted date-time indicating when the rotation starts.
- **recurrence_type** (string, default: "weekly"): The type of recurrence for the rotation (options: "daily", "weekly", "monthly").
- **participants** (array, required): A list of user identifiers who participate in the rotation.
- **rotation_length** (integer, optional): The length of time each participant is on call, in days.

#### Best Practices

1. **Time Zone Consistency**: Always specify the `timezone` to ensure all participants are aware of the time context.
2. **Structured Rotation**: Use meaningful names for rotations, and schedule computations regularly to reflect reality.
3. **Clear Naming**: Rotation names should be descriptive, indicating their purpose or frequency.

#### Example

```yaml
schedule_id: "team-ny-support-schedule"
name: "New York Support Schedule"
timezone: "America/New_York"
rotations:
  - name: "Weekdays"
    start_time: "2023-01-01T09:00:00-05:00"
    recurrence_type: "weekly"
    participants:
      - "user123"
      - "user456"
    rotation_length: 7
```

---

## Escalation Policy Configuration

### escalation-policy-config.yaml

The `escalation-policy-config.yaml` file defines the procedures and hierarchies for escalating incidents. This configuration ensures that unresolved incidents receive the attention they require by notifying the appropriate team members progressively.

#### Fields

- **policy_id** (string, required): A unique identifier for the escalation policy.
- **name** (string, required): A descriptive name for the escalation policy.
- **rules** (array, required): List of escalation rules to be followed.

##### Escalation Rule Configuration

Each rule consists of:

- **level** (integer, required): The priority level of the escalation, where lower numbers indicate initial response.
- **notify** (array, required): List of user identifiers or group names to notify.
- **time_delay** (string, required): ISO 8601 formatted duration after which the next level should be triggered if not acknowledged.

#### Best Practices

1. **Clear Escalation Paths**: Ensure that each escalation step has a clear point of contact.
2. **Timeliness**: Design `time_delay` values thoughtfully to balance between allowing response time and urgency.

#### Example

```yaml
policy_id: "critical_incident_policy"
name: "Critical Incident Escalation Policy"
rules:
  - level: 1
    notify:
      - "user123"
    time_delay: "PT15M"
  - level: 2
    notify:
      - "team-leads"
    time_delay: "PT30M"
  - level: 3
    notify:
      - "management"
    time_delay: "PT1H"
```

---

## Notification Rules Configuration

### notification-rules-config.yaml

The `notification-rules-config.yaml` is crafted to establish methods and channels through which communication occurs during incidents. Notifications can be tailored based on the severity and type of incident.

#### Fields

- **rule_id** (string, required): A unique identifier for the notification rule.
- **triggers** (array, required): Conditions that activate the notification.
- **channels** (array, required): Channels through which notifications are dispatched (e.g., email, SMS, Slack).

##### Notification Channel Configuration

Specify how notifications are sent via:

- **type** (string, required): The type of channel (options: "email", "sms", "slack").
- **config** (object, required): Channel-specific configuration settings.

#### Best Practices

1. **Coverage**: Ensure notification channels cover all critical communication paths.
2. **Redundancy**: Utilize multiple channels to guarantee delivery in case one fails.

#### Example

```yaml
rule_id: "high_severity_notification"
triggers:
  - "incident_created"
  - "incident_reopened"
channels:
  - type: "email"
    config:
      smtp_server: "smtp.example.com"
      from_address: "alerts@example.com"
  - type: "slack"
    config:
      webhook_url: "https://hooks.slack.com/services/..."
```

---

## Integration Configuration

### integration-config.yaml

The `integration-config.yaml` details the setup for integrating Jira JSM On-Call with external tools and services, enhancing functionality and automating workflows.

#### Fields

- **integration_id** (string, required): A unique identifier for the integration.
- **type** (string, required): The type of integration (options: "webhook", "api", "plugin").
- **settings** (object, required): Integration-specific configuration parameters.

##### Integration Type Configuration

Each type of integration may require different settings, such as endpoints, authentication details, and payload specifications.

#### Best Practices

1. **Security**: Configure secure and authenticated connections for all integrations.
2. **Testing**: Validate integrations in a non-production environment before official deployment.

#### Example

```yaml
integration_id: "pagerduty_integration"
type: "api"
settings:
  api_key: "your_api_key_here"
  base_url: "https://api.pagerduty.com"
  incident_handler: "create_incident"
```

---

## Conclusion

Configuring Jira JSM On-Call efficiently requires a detailed understanding of its various configurations. This documentation aims to provide a comprehensive reference to assist you in maximizing the value of your on-call management setup. Remember to regularly review and update configurations to adapt to changing organizational requirements, ensuring that best practices are followed for optimal operational efficiency.