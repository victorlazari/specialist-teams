# Configuration Schemas for Support Systems: Jira JSM Workflows, SLA Timers, PagerDuty Escalation Policies, and Datadog Alert Thresholds

---

## Table of Contents

1. [Introduction](#introduction)  
2. [Jira JSM Workflow Configuration Schema](#jira-jsm-workflow-configuration-schema)  
   - 2.1 Overview  
   - 2.2 JSON Schema Example  
   - 2.3 Key Considerations for Production and Worst-Case Scenarios  
3. [SLA Timers Configuration Schema](#sla-timers-configuration-schema)  
   - 3.1 SLA Components  
   - 3.2 YAML Schema Example  
   - 3.3 Handling SLA Breaches in Critical Incidents  
4. [PagerDuty Escalation Policies Configuration Schema](#pagerduty-escalation-policies-configuration-schema)  
   - 4.1 Overview and Best Practices  
   - 4.2 JSON Schema Example  
   - 4.3 Escalation Strategy for High-Severity Incidents  
5. [Datadog Alert Thresholds Configuration Schema](#datadog-alert-thresholds-configuration-schema)  
   - 5.1 Alert Types and Thresholds  
   - 5.2 YAML Schema Example  
   - 5.3 Threshold Tuning in Production and Disaster Scenarios  
6. [Integration with Other Specialist-Teams Repository Files](#integration-with-other-specialist-teams-repository-files)  
7. [Summary and Best Practices](#summary-and-best-practices)  

---

## Introduction

In high-pressure **production operations** environments, well-defined configuration schemas for support systems are critical to ensuring timely issue detection, escalation, and resolution. This document provides a **detailed, practical**, and **comprehensive** overview of configuration schemas for four core support systems widely used in technical support operations:

- **Jira Service Management (JSM) Workflows**  
- **SLA Timers**  
- **PagerDuty Escalation Policies**  
- **Datadog Alert Thresholds**  

Each section presents schema examples in JSON or YAML, emphasizing **clarity, scalability, and worst-case readiness**. The schemas are designed to enable automation, reduce human error, and support rapid incident lifecycle management.

---

## Jira JSM Workflow Configuration Schema

### 2.1 Overview

Jira Service Management workflows define the lifecycle of support tickets. A robust workflow schema allows for:

- Clear **state transitions** with permissions and conditions  
- Automated **status updates** based on SLA timers and alerts  
- Integration hooks for **notification** and external system triggers  
- Escalations embedded within the workflow  

In production and worst-case scenarios, workflows must prevent tickets from becoming "stuck" and ensure clear visibility on escalating issues.

---

### 2.2 JSON Schema Example

```json
{
  "workflowName": "Production Incident Workflow",
  "statuses": [
    "New",
    "Acknowledged",
    "In Progress",
    "Waiting on Customer",
    "Escalated",
    "Resolved",
    "Closed"
  ],
  "transitions": [
    {
      "name": "Acknowledge Ticket",
      "from": "New",
      "to": "Acknowledged",
      "conditions": [
        {"type": "permission", "value": "Support Agent"},
        {"type": "ticketPriority", "value": ["High", "Critical"]}
      ],
      "postFunctions": [
        {"type": "setField", "field": "assignee", "value": "currentUser"},
        {"type": "triggerNotification", "template": "acknowledge_email"}
      ]
    },
    {
      "name": "Start Work",
      "from": "Acknowledged",
      "to": "In Progress",
      "conditions": [
        {"type": "permission", "value": "Support Agent"}
      ],
      "validators": [
        {"type": "customFieldNotEmpty", "field": "Root Cause Analysis"}
      ]
    },
    {
      "name": "Escalate Issue",
      "from": ["In Progress", "Acknowledged"],
      "to": "Escalated",
      "conditions": [
        {"type": "slaBreached", "slaId": "P1_Response_SLA"}
      ],
      "postFunctions": [
        {"type": "addComment", "comment": "Escalation triggered due to SLA breach."},
        {"type": "triggerExternalWebhook", "url": "https://pagerduty.example.com/api/v1/escalate"}
      ]
    },
    {
      "name": "Resolve Ticket",
      "from": ["In Progress", "Escalated"],
      "to": "Resolved",
      "conditions": [
        {"type": "permission", "value": ["Support Agent", "Support Manager"]}
      ],
      "validators": [
        {"type": "customFieldNotEmpty", "field": "Resolution"}
      ],
      "postFunctions": [
        {"type": "triggerNotification", "template": "resolution_email"}
      ]
    },
    {
      "name": "Close Ticket",
      "from": "Resolved",
      "to": "Closed",
      "conditions": [
        {"type": "permission", "value": "Support Manager"}
      ]
    }
  ],
  "permissions": {
    "Support Agent": ["Acknowledge Ticket", "Start Work", "Resolve Ticket"],
    "Support Manager": ["Close Ticket", "Resolve Ticket", "Escalate Issue"]
  }
}
```

---

### 2.3 Key Considerations for Production and Worst-Case Scenarios

- **SLA Breach Handling:** Automatic transitions to "Escalated" status when SLA timers breach prevent delays in critical tickets.  
- **Role-based Permissions:** Enforce strict permission checks to avoid unauthorized status changes.  
- **Mandatory Fields:** Validators ensure critical data such as root cause and resolution are always populated.  
- **Notification Hooks:** Post-functions trigger emails and external integrations (e.g., PagerDuty) automatically.  
- **Multi-Origin Transitions:** Allow escalation from multiple states to handle diverse workflows.  
- **Audit Trails:** Every transition should log timestamps and user actions to maintain accountability.

---

## SLA Timers Configuration Schema

### 3.1 SLA Components

An SLA timer defines the expected timeframes for ticket response and resolution, often tied to priority levels. Key elements include:

- **SLA ID and Name**  
- **Applicable Priorities**  
- **Response and Resolution Goals** (in minutes/hours)  
- **Working Hours** (business hours vs 24/7 support)  
- **Pause Conditions** (e.g., waiting for customer input)  
- **Breach Actions** (notifications, escalations)  

---

### 3.2 YAML Schema Example

```yaml
slaTimers:
  - slaId: P1_Response_SLA
    name: "Priority 1 Response Time"
    priorities: ["Critical", "High"]
    responseTimeMinutes: 15
    resolutionTimeMinutes: 120
    workingHours:
      start: "08:00"
      end: "20:00"
      timezone: "UTC"
      daysOfWeek: ["Monday", "Tuesday", "Wednesday", "Thursday", "Friday"]
    pauseConditions:
      - status: "Waiting on Customer"
      - status: "Pending Vendor"
    breachActions:
      - type: "sendNotification"
        recipients: ["support_manager", "on_call_engineer"]
        template: "sla_breach_notification"
      - type: "triggerEscalation"
        policyId: "pagerduty_p1_escalation"
  - slaId: P3_Response_SLA
    name: "Priority 3 Response Time"
    priorities: ["Medium"]
    responseTimeMinutes: 120
    resolutionTimeMinutes: 1440
    workingHours:
      start: "09:00"
      end: "17:00"
      timezone: "UTC"
      daysOfWeek: ["Monday", "Tuesday", "Wednesday", "Thursday", "Friday"]
    pauseConditions: []
    breachActions:
      - type: "sendNotification"
        recipients: ["support_lead"]
        template: "sla_breach_notification"
```

---

### 3.3 Handling SLA Breaches in Critical Incidents

- **Pause Conditions:** Pausing timers during "Waiting on Customer" or "Pending Vendor" statuses avoids unfair breaches.  
- **Multiple Breach Actions:** Combine notifications and automated escalations to reduce manual oversight risk.  
- **Timezone Awareness:** Correctly configured working hours prevent false SLA breaches outside business time.  
- **Escalation Linkage:** SLA breaches should trigger PagerDuty policies or Jira escalations to ensure rapid response.  
- **Priority Differentiation:** Tighter SLAs for P1/P2 incidents reflect their criticality in production environments.

---

## PagerDuty Escalation Policies Configuration Schema

### 4.1 Overview and Best Practices

PagerDuty policies define how alerts and incidents escalate through on-call schedules. Key features include:

- **Escalation Rules:** Define escalation targets and wait times before escalating.  
- **Multiple Escalation Levels:** Support multi-step escalation chains.  
- **On-Call Schedules:** Integrate dynamic schedules for availability.  
- **Auto-Resolve and Reassignment:** Policies for incident lifecycle management.  
- **High Availability:** Redundancy in escalation paths to mitigate single points of failure.

---

### 4.2 JSON Schema Example

```json
{
  "policyId": "pagerduty_p1_escalation",
  "name": "P1 Incident Escalation Policy",
  "description": "Escalates critical incidents through multiple on-call layers",
  "escalationRules": [
    {
      "escalationLevel": 1,
      "targets": [
        {"type": "user", "id": "user_12345"},
        {"type": "schedule", "id": "schedule_primary_oncall"}
      ],
      "escalationDelayMinutes": 10
    },
    {
      "escalationLevel": 2,
      "targets": [
        {"type": "user", "id": "user_67890"},
        {"type": "schedule", "id": "schedule_secondary_oncall"}
      ],
      "escalationDelayMinutes": 15
    },
    {
      "escalationLevel": 3,
      "targets": [
        {"type": "user", "id": "support_manager"},
        {"type": "schedule", "id": "schedule_manager_oncall"}
      ],
      "escalationDelayMinutes": 20
    }
  ],
  "autoResolveTimeoutMinutes": 4320,
  "reassignmentEnabled": true,
  "reassignmentTimeoutMinutes": 30
}
```

---

### 4.3 Escalation Strategy for High-Severity Incidents

- **Short Escalation Delays:** Critical incidents require rapid escalation (e.g., 10 minutes to first responder).  
- **Multi-Channel Targets:** Both individual users and schedules ensure coverage even when specific people are unavailable.  
- **Reassignment:** Automatically reassign if no acknowledgment within a timeout reduces incident stagnation.  
- **Auto-Resolve:** Long-running incidents can auto-resolve after a defined timeout to clean up stale alerts but must be balanced to prevent premature closure.  
- **Redundancy:** Escalation chains should include managerial roles to ensure oversight on prolonged incidents.

---

## Datadog Alert Thresholds Configuration Schema

### 5.1 Alert Types and Thresholds

Datadog monitors system metrics, triggering alerts when thresholds are breached. Alerts can be:

- **Metric Threshold Alerts:** Triggered when a metric crosses a static or dynamic threshold.  
- **Anomaly Detection Alerts:** Triggered on unusual metric patterns.  
- **Composite Alerts:** Logical combinations of other alerts.  

Important parameters:

- **Metric Name**  
- **Threshold Values** (warning, critical)  
- **Evaluation Window** (e.g., over last 5 minutes)  
- **Notification Channels**  
- **Tags for Filtering**  
- **Recovery Thresholds**  

---

### 5.2 YAML Schema Example

```yaml
datadogAlerts:
  - alertId: cpu_high_usage
    name: "CPU High Usage Alert"
    metric: system.cpu.user
    thresholds:
      warning: 70
      critical: 90
    evaluationWindowMinutes: 5
    notifyChannels:
      - email: "on_call_engineer@example.com"
      - pagerduty: "pagerduty_p1_escalation"
    tags:
      environment: production
      service: web_frontend
    recoveryThreshold: 65
    alertType: "metric_threshold"

  - alertId: disk_io_anomaly
    name: "Disk I/O Anomaly Alert"
    metric: system.disk.io.avg_time
    alertType: "anomaly_detection"
    evaluationWindowMinutes: 10
    notifyChannels:
      - email: "support_team@example.com"
    tags:
      environment: production
      service: database
```

---

### 5.3 Threshold Tuning in Production and Disaster Scenarios

- **Dynamic Thresholding:** Where possible, use anomaly detection to reduce false positives from transient spikes.  
- **Warning vs Critical:** Two-level thresholds enable early warning without overwhelming teams.  
- **Recovery Thresholds:** Define hysteresis to avoid alert flapping.  
- **Tagging:** Use rich tags for filtering and routing alerts appropriately.  
- **Notification Integration:** Alerts must integrate directly with PagerDuty escalation policies for seamless incident creation.  
- **Evaluation Window:** Short windows catch fast incidents but beware of noise; balance is essential.  
- **Environment Separation:** Alerts must be scoped correctly to avoid alert fatigue from non-production environments.

---

## Integration with Other Specialist-Teams Repository Files

This configuration schema document is a **core component** of the broader support operations ecosystem managed in the `specialist-teams` repository. It closely interacts with the following six files:

| File Name                                | Relation to This Schema                                            |
|------------------------------------------|-------------------------------------------------------------------|
| `01-incident-response-playbook.md`       | Defines step-by-step processes triggered by Jira workflow states and PagerDuty escalations configured here. |
| `12-communication-templates.md`           | Templates referenced in Jira post-functions and SLA breach notifications. |
| `23-oncall-schedules.yaml`                 | Source schedules referenced in PagerDuty escalation policies.    |
| `34-monitoring-metrics-definitions.md`    | Defines metrics whose thresholds are configured in Datadog alerts here. |
| `38-postmortem-templates.md`                | Utilizes data collected through Jira workflows and SLA breaches to document incidents. |
| `42-support-training-guidelines.md`         | Training materials reference these schemas to educate new support staff on operational tooling and escalation protocols. |

**Key integration points:**

- **Automated Triggering:** Jira workflows invoke PagerDuty policies and send SLA breach notifications using templates defined in communication files.  
- **Data Consistency:** Metrics definitions guide which Datadog alerts to set thresholds on, aligning monitoring with incident triggers.  
- **Operational Readiness:** On-call schedules linked in PagerDuty policies ensure coverage aligning with documented support rotations.  
- **Feedback Loop:** Postmortem documentation leverages ticket histories and SLA breach data to improve policies and workflows.  
- **Training Alignment:** Staff learn operational procedures based on these schemas, ensuring consistent interpretation and usage.

---

## Summary and Best Practices

While this document refrains from providing a brief summary per requirements, it is critical to highlight **best practices** embedded in these schemas for production tech support:

- **Automation first:** Use automated transitions, notifications, and escalations to minimize human error and speed up responses.  
- **Clear role definitions:** Enforce permissions rigorously in workflows and escalation policies.  
- **Prioritize critical incidents:** Tighter SLAs, rapid escalation, and focused alert thresholds ensure that P1/P2 issues receive appropriate attention.  
- **Fail-safe designs:** Include pause conditions, recovery thresholds, and redundant escalation paths to avoid failure in worst-case scenarios.  
- **Integration and consistency:** Align configuration with other repository assets like playbooks and communication templates for an end-to-end support lifecycle.  
- **Continuous improvement:** Use data from workflows and alerts to refine thresholds, escalations, and training continuously.

These schemas form the backbone of resilient, scalable, and effective tech support operations that can withstand the pressures of production environments and worst-case incident management.

---

# End of Document