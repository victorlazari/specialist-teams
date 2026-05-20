# Tech Support Operations: Comprehensive Reference Guide

## 1. Introduction

In the fast-paced environment of modern technology operations, the ability to rapidly diagnose, triage, and resolve incidents is paramount. Tech Support Operations (TechOps) specialists serve as the critical bridge between customer-facing support teams and backend engineering. They are responsible for maintaining system reliability, managing incidents, and ensuring that support workflows are optimized for speed and accuracy.

This comprehensive reference guide is designed to equip TechOps specialists with the knowledge and tools necessary to excel in their roles. It covers a wide array of essential support tools, including Jira Service Management (JSM), Zendesk, PagerDuty, and Datadog. Furthermore, it delves into advanced logging queries using Splunk and the ELK stack, providing practical examples for real-world scenarios. Finally, it explores the creation and implementation of automated runbooks, which are crucial for standardizing incident response and reducing mean time to resolution (MTTR).

By mastering the concepts and techniques outlined in this guide, TechOps professionals can significantly enhance their operational efficiency, minimize downtime, and deliver exceptional support experiences.

## 2. Support Tools Reference

The modern TechOps ecosystem relies on a diverse set of tools to manage tickets, monitor systems, and coordinate incident response. Understanding the capabilities and integrations of these tools is essential for effective operations.

### 2.1 Jira Service Management (JSM)

Jira Service Management (JSM) is a powerful IT service management (ITSM) solution that enables teams to manage incidents, problems, changes, and requests. It integrates seamlessly with Jira Software, allowing for smooth collaboration between support and development teams.

**Key Features:**
- **Incident Management:** Streamlined workflows for logging, tracking, and resolving incidents.
- **Service Request Management:** Customizable portals for users to submit requests.
- **Change Management:** Structured processes for planning, approving, and implementing changes.
- **Asset Management:** Tracking and managing IT assets and their relationships.

**Best Practices for TechOps:**
- **Automate Triage:** Use automation rules to automatically assign tickets based on keywords, components, or issue types.
- **Integrate with Monitoring:** Connect JSM with monitoring tools like Datadog to automatically create incidents when alerts are triggered.
- **Standardize Workflows:** Define clear statuses and transitions to ensure consistent ticket handling.

**Example JQL (Jira Query Language) Queries:**
- Find all open critical incidents: `project = "ITSM" AND issuetype = Incident AND priority = Critical AND status != Closed`
- Find tickets assigned to the current user that are breached SLA: `assignee = currentUser() AND "Time to resolution" = breached()`

### 2.2 Zendesk

Zendesk is a leading customer service platform that provides a unified workspace for managing customer interactions across various channels, including email, chat, and social media.

**Key Features:**
- **Omnichannel Support:** Centralized management of customer inquiries from multiple sources.
- **Knowledge Base:** Integrated self-service portals for customers to find answers independently.
- **Macros and Triggers:** Automation tools for streamlining repetitive tasks and responses.
- **Reporting and Analytics:** Comprehensive dashboards for tracking support metrics.

**Best Practices for TechOps:**
- **Utilize Macros:** Create macros for common issues to ensure consistent and rapid responses.
- **Implement Triggers:** Set up triggers to automatically escalate tickets based on specific conditions, such as time elapsed or customer priority.
- **Leverage the API:** Use the Zendesk API to integrate with internal tools and automate data synchronization.

**Example Zendesk API Request (cURL):**
```bash
curl https://{subdomain}.zendesk.com/api/v2/tickets.json \
  -v -u {email_address}:{password} \
  -X POST -d '{"ticket": {"subject": "My printer is on fire!", "comment": { "body": "The smoke is very colorful." }}}' \
  -H "Content-Type: application/json"
```

### 2.3 PagerDuty

PagerDuty is an incident management platform that provides reliable alerting, on-call scheduling, and automated escalation policies. It is essential for ensuring that the right people are notified promptly when critical issues arise.

**Key Features:**
- **On-Call Management:** Flexible scheduling and rotation management for on-call teams.
- **Escalation Policies:** Automated routing of alerts to the appropriate personnel based on predefined rules.
- **Incident Response:** Tools for coordinating response efforts, including conference bridges and status updates.
- **Analytics:** Insights into team performance and system reliability.

**Best Practices for TechOps:**
- **Define Clear Escalation Paths:** Ensure that escalation policies are well-defined and up-to-date to prevent missed alerts.
- **Use Urgency Levels:** Differentiate between high-urgency and low-urgency alerts to minimize alert fatigue.
- **Integrate with ChatOps:** Connect PagerDuty with Slack or Microsoft Teams to facilitate communication during incidents.

**Example PagerDuty CLI Command:**
```bash
# Acknowledge an incident
pd incident ack -i INCIDENT_ID

# Resolve an incident
pd incident resolve -i INCIDENT_ID -m "Resolved after restarting the service."
```

### 2.4 Datadog

Datadog is a comprehensive monitoring and analytics platform that provides visibility into infrastructure, applications, and logs. It is a critical tool for identifying performance bottlenecks and diagnosing system failures.

**Key Features:**
- **Infrastructure Monitoring:** Real-time metrics for servers, containers, and cloud services.
- **Application Performance Monitoring (APM):** Tracing and profiling for distributed applications.
- **Log Management:** Centralized collection, parsing, and analysis of logs.
- **Dashboards and Alerts:** Customizable visualizations and automated notifications based on metric thresholds.

**Best Practices for TechOps:**
- **Create Comprehensive Dashboards:** Build dashboards that provide a holistic view of system health, including key performance indicators (KPIs) and error rates.
- **Set Meaningful Alerts:** Configure alerts that trigger only when actionable issues occur, avoiding false positives.
- **Correlate Metrics and Logs:** Use Datadog's integration capabilities to correlate metrics with corresponding logs for faster root cause analysis.

**Example Datadog Monitor Configuration (JSON):**
```json
{
  "name": "High CPU Utilization on Web Servers",
  "type": "metric alert",
  "query": "avg(last_5m):avg:system.cpu.idle{role:web} < 20",
  "message": "CPU utilization is critically high on web servers. @pagerduty-Web_Team",
  "tags": ["env:production", "team:web"],
  "options": {
    "notify_audit": false,
    "locked": false,
    "timeout_h": 0,
    "new_host_delay": 300,
    "require_full_window": true,
    "notify_no_data": false,
    "renotify_interval": 0,
    "escalation_message": "",
    "no_data_timeframe": null,
    "include_tags": true,
    "thresholds": {
      "critical": 20,
      "warning": 30
    }
  }
}
```

## 3. Logging Queries and Analysis

Effective log analysis is a cornerstone of TechOps. Logs provide detailed records of system events, errors, and user activities, making them invaluable for troubleshooting and security investigations.

### 3.1 Splunk Search Processing Language (SPL)

Splunk is a powerful platform for searching, monitoring, and analyzing machine-generated data. Its Search Processing Language (SPL) is a robust tool for extracting insights from massive volumes of logs.

**Basic SPL Concepts:**
- **Search Terms:** Keywords, phrases, or field-value pairs used to filter data.
- **Commands:** Instructions that tell Splunk what to do with the search results (e.g., `stats`, `timechart`, `eval`).
- **Pipes (`|`):** Used to pass the output of one command as the input to the next.

**Common SPL Queries for TechOps:**

1. **Find all errors in a specific application:**
   ```spl
   index=production sourcetype=app_logs level=ERROR OR level=FATAL
   ```

2. **Count the number of errors by host:**
   ```spl
   index=production level=ERROR | stats count by host | sort - count
   ```

3. **Calculate the average response time for an API endpoint:**
   ```spl
   index=production sourcetype=api_access uri_path="/api/v1/users" | stats avg(response_time) as avg_response_time
   ```

4. **Identify IP addresses with the most failed login attempts:**
   ```spl
   index=security action=failure | stats count by src_ip | sort - count | head 10
   ```

5. **Create a timechart of HTTP status codes:**
   ```spl
   index=web sourcetype=access_combined | timechart count by status
   ```

### 3.2 Elasticsearch/Logstash/Kibana (ELK)

The ELK stack is a popular open-source solution for log management and analysis. It consists of Elasticsearch (search and analytics engine), Logstash (data processing pipeline), and Kibana (visualization platform).

**Kibana Query Language (KQL):**
KQL is used to filter data in Kibana dashboards and Discover views. It provides a simple syntax for searching fields and values.

**Common KQL Queries for TechOps:**

1. **Find all logs with a specific status code:**
   ```kql
   response.status: 500
   ```

2. **Search for errors in a specific service:**
   ```kql
   service.name: "payment-gateway" AND log.level: "ERROR"
   ```

3. **Find requests that took longer than 1000ms:**
   ```kql
   http.response.duration > 1000
   ```

4. **Search for a specific user ID across all logs:**
   ```kql
   user.id: "usr_12345abcde"
   ```

**Elasticsearch Query DSL (JSON):**
For more complex queries, Elasticsearch provides a robust JSON-based Query DSL.

**Example Query DSL: Find all 5xx errors in the last 24 hours:**
```json
{
  "query": {
    "bool": {
      "must": [
        {
          "range": {
            "response.status": {
              "gte": 500,
              "lt": 600
            }
          }
        },
        {
          "range": {
            "@timestamp": {
              "gte": "now-24h",
              "lte": "now"
            }
          }
        }
      ]
    }
  }
}
```

## 4. Automated Runbooks and Incident Response

Runbooks are documented procedures that outline the steps required to resolve specific types of incidents. Automating these runbooks is a critical strategy for reducing MTTR and minimizing human error during high-stress situations.

### 4.1 Runbook Design Principles

When designing automated runbooks, adhere to the following principles:

- **Idempotency:** Ensure that executing the runbook multiple times has the same effect as executing it once. This prevents unintended consequences if a runbook is triggered accidentally or retried.
- **Modularity:** Break down complex procedures into smaller, reusable components. This makes runbooks easier to maintain and update.
- **Clear Logging:** Implement comprehensive logging within the runbook to track its execution and identify any failures.
- **Human-in-the-Loop (HITL):** For critical actions, such as restarting a primary database, include a manual approval step before proceeding.
- **Continuous Improvement:** Regularly review and update runbooks based on post-incident reviews (PIRs) and changing system architectures.

### 4.2 Sample Automated Runbook: High CPU Utilization

**Scenario:** A monitoring alert indicates that a web server is experiencing sustained CPU utilization above 90%.

**Trigger:** Datadog alert webhook.

**Automated Steps:**

1. **Acknowledge Alert:** Automatically acknowledge the PagerDuty incident to inform the team that the runbook is executing.
2. **Gather Diagnostics:**
   - Connect to the affected server via SSH.
   - Execute `top -b -n 1` to identify the processes consuming the most CPU.
   - Execute `dmesg | tail -n 50` to check for kernel errors.
   - Capture the output and append it to the Jira incident ticket.
3. **Attempt Remediation:**
   - If the high CPU is caused by a known background worker process, attempt to gracefully restart the service (`systemctl restart worker-service`).
4. **Verify Remediation:**
   - Wait 60 seconds.
   - Check the CPU utilization metric via the Datadog API.
5. **Escalate or Resolve:**
   - If CPU utilization has returned to normal levels (< 70%), resolve the PagerDuty incident and update the Jira ticket.
   - If CPU utilization remains high, escalate the PagerDuty incident to the secondary on-call engineer and add a comment to the Jira ticket indicating that automated remediation failed.

**Example Python Script for Diagnostics (Snippet):**
```python
import paramiko
import requests

def gather_diagnostics(hostname, username, key_filename):
    ssh = paramiko.SSHClient()
    ssh.set_missing_host_key_policy(paramiko.AutoAddPolicy())
    
    try:
        ssh.connect(hostname, username=username, key_filename=key_filename)
        
        # Get top processes
        stdin, stdout, stderr = ssh.exec_command("top -b -n 1 | head -n 20")
        top_output = stdout.read().decode('utf-8')
        
        return top_output
    except Exception as e:
        return f"Error connecting to {hostname}: {str(e)}"
    finally:
        ssh.close()

# Example usage
# output = gather_diagnostics("web-server-01", "ubuntu", "/path/to/key.pem")
# update_jira_ticket("INC-123", output)
```

### 4.3 Sample Automated Runbook: Database Connection Failure

**Scenario:** The application is unable to connect to the primary PostgreSQL database, resulting in a surge of 500 errors.

**Trigger:** Application APM alert (e.g., high error rate for database queries).

**Automated Steps:**

1. **Create Major Incident:** Automatically create a high-priority incident in JSM and page the Database Administration (DBA) team via PagerDuty.
2. **Check Database Status:**
   - Query the cloud provider API (e.g., AWS RDS) to check the status of the database instance.
   - Attempt a basic connection test using `psql` or a simple script.
3. **Analyze Logs:**
   - Query Splunk or ELK for recent database error logs (e.g., "connection refused", "too many clients").
   - Append the top 10 most frequent errors to the incident ticket.
4. **Evaluate Failover (HITL):**
   - If the primary database is unreachable, prepare a failover procedure.
   - Send a notification to the incident channel (e.g., Slack) requesting manual approval to initiate the failover to the read replica.
5. **Execute Failover (Upon Approval):**
   - Execute the cloud provider API command to promote the read replica to primary.
   - Update application configuration (if necessary) to point to the new primary endpoint.
6. **Post-Incident Tasks:**
   - Once the application is stable, generate a preliminary incident report and schedule a post-mortem meeting.

## 5. CLI and API Integration Examples

Command-Line Interfaces (CLIs) and Application Programming Interfaces (APIs) are essential tools for TechOps specialists, enabling them to automate tasks and interact with systems programmatically.

### 5.1 JSM CLI

While Atlassian does not provide an official, comprehensive CLI for JSM, various open-source tools and REST API wrappers exist. Using the REST API directly via `curl` or Python is often the most flexible approach.

**Example: Create a Jira Issue via REST API (cURL):**
```bash
curl -u email@example.com:API_TOKEN \
  -X POST \
  -H "Content-Type: application/json" \
  --data '{
    "fields": {
       "project": {
          "key": "ITSM"
       },
       "summary": "Automated Incident: High Latency",
       "description": "Monitoring has detected high latency on the payment gateway.",
       "issuetype": {
          "name": "Incident"
       }
   }
}' \
  "https://your-domain.atlassian.net/rest/api/3/issue"
```

### 5.2 PagerDuty CLI

The PagerDuty CLI (`pd`) is a powerful tool for managing incidents directly from the terminal.

**Installation (macOS/Linux):**
```bash
# Download the binary and move it to your PATH
curl -L https://github.com/PagerDuty/go-pd/releases/latest/download/pd-linux-amd64 -o pd
chmod +x pd
sudo mv pd /usr/local/bin/
```

**Configuration:**
```bash
pd login
# Follow the prompts to enter your API token
```

**Common Commands:**
- `pd incident list`: List open incidents.
- `pd incident ack -i <ID>`: Acknowledge an incident.
- `pd incident resolve -i <ID>`: Resolve an incident.
- `pd oncall`: View the current on-call schedule.

## 6. Conclusion

Tech Support Operations is a dynamic and demanding field that requires a deep understanding of various tools, systems, and processes. By mastering platforms like JSM, Zendesk, PagerDuty, and Datadog, and by leveraging the power of log analysis and automated runbooks, TechOps specialists can significantly improve incident response times and ensure the reliability of critical services.

This reference guide serves as a foundational resource, providing practical examples and best practices that can be applied to real-world scenarios. Continuous learning and adaptation are essential in this ever-evolving landscape, and the techniques outlined here will empower TechOps professionals to meet the challenges of modern technology operations with confidence and expertise.

---
*Note: This file is part of the specialist-teams repository. It serves as the comprehensive reference for Tech Support Operations, complementing the other specialist profiles by providing the technical depth required for advanced incident management and tool integration.*
