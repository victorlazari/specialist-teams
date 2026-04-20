# Ticket System Reports — Platform Menu Catalog

> Master catalog of reports to be implemented in the platform.
> Organized as a navigable menu: **Category → Report → Description + Consolidation + Filters**.
> Ordered from most-used in the market to least-used (based on ITSM / CS industry adoption: Jira, ServiceNow, Zendesk, Freshdesk, Salesforce Service Cloud patterns).

---

## Table of Contents

1. [Operational Status & Workflow](#1-operational-status--workflow) — universal
2. [SLA & Response Time](#2-sla--response-time) — near-universal
3. [Volume & Trends](#3-volume--trends)
4. [Agent & Team Performance](#4-agent--team-performance)
5. [Customer & Requester](#5-customer--requester)
6. [Backlog & Aging](#6-backlog--aging)
7. [Quality & Escalation](#7-quality--escalation)
8. [Incident Management](#8-incident-management)
9. [Security & Compliance](#9-security--compliance)
10. [Financial & Business Impact](#10-financial--business-impact)
11. [Advanced Analytics & Forecasting](#11-advanced-analytics--forecasting)

---

## Common Filters (Available on Almost Every Report)

These are **global filters** applicable across most reports — listed once here; individual reports list only *additional* or *report-specific* filters.

- **Date Range** — `Today`, `Yesterday`, `Last 7 / 30 / 90 days`, `This / Last Week`, `This / Last Month`, `This / Last Quarter`, `YTD`, `Custom range (from–to)`
- **Organization / Tenant** (multi-tenant installs)
- **Project / Product / Workspace**
- **Department / Business Unit**
- **Team / Queue / Group**
- **Assignee / Agent**
- **Reporter / Requester / Customer**
- **Priority** — `Highest, High, Medium, Low, Lowest`
- **Severity** — `S1, S2, S3, S4`
- **Status** — `Open, In Progress, Pending, Resolved, Closed, Reopened, Cancelled`
- **Status Category** — `To Do, In Progress, Done`
- **Ticket Type** — `Incident, Request, Problem, Change, Task, Bug, Feature`
- **Channel / Source** — `Email, Chat, Phone, Web Form, API, Slack, Portal`
- **Component / Service / Product Area**
- **Labels / Tags**
- **Custom Fields** (any)
- **Language / Locale**
- **Region / Country / Timezone**
- **SLA Policy**
- **Group By** (pivot dimension)
- **Sort By** (column + direction)

---

# 1. Operational Status & Workflow

*The foundation reports — every ticketing platform offers these. Highest daily consumption.*

- **Tickets by Status**
  - *Description:* Distribution of tickets across their current workflow status (Open, In Progress, Pending, Resolved, Closed, etc.).
  - *Consolidation:* `COUNT(tickets) GROUP BY status`. Snapshot of current state; typically served as a bar/donut chart + table. Can be trended by taking daily snapshots into a history table.
  - *Filters:* Status (multi-select), Status Category, Ticket Type, Priority, Assignee, Team, Project, Date Range (created/updated), Channel, Custom Fields.

- **Tickets by Priority**
  - *Description:* Volume distribution by priority level to guide triage and resource allocation.
  - *Consolidation:* `COUNT(tickets) GROUP BY priority` on a filtered scope. Usually rendered stacked by status.
  - *Filters:* Priority, Status, Ticket Type, Assignee, Team, Date Range, Project, Severity, SLA Policy, Channel.

- **Tickets by Type / Category**
  - *Description:* Breakdown by issue type (Incident, Request, Problem, Change, Bug, Feature, etc.) to understand workload mix.
  - *Consolidation:* `COUNT(tickets) GROUP BY ticket_type` plus cross-tab with status/priority.
  - *Filters:* Ticket Type, Sub-type, Category, Sub-category, Status, Priority, Date Range, Project, Team, Source.

- **Unassigned Tickets**
  - *Description:* Open tickets with no assignee — queue of work awaiting ownership.
  - *Consolidation:* Real-time list query: `assignee IS NULL AND status_category != Done`. Rendered as actionable table with age column.
  - *Filters:* Queue, Team, Priority, Status, Ticket Type, Channel, Created Date, Age (hours/days), SLA at risk flag.

- **Tickets by Assignee**
  - *Description:* Current workload per agent — who is working on what.
  - *Consolidation:* `COUNT(tickets) GROUP BY assignee WHERE status_category != Done` with drill-down to ticket list.
  - *Filters:* Assignee (multi), Team, Priority, Status, Ticket Type, Project, Workload Threshold, Availability Status.

- **Tickets by Team / Queue**
  - *Description:* Volume and state of tickets grouped by support team or queue.
  - *Consolidation:* `COUNT(tickets) GROUP BY team, status` — often pivoted as a matrix.
  - *Filters:* Team, Sub-team, Queue, Status, Priority, Date Range, Ticket Type, Region.

- **Tickets by Channel / Source**
  - *Description:* Where tickets are originating from (email, chat, phone, portal, API, social).
  - *Consolidation:* `COUNT(tickets) GROUP BY channel` trended over time.
  - *Filters:* Channel, Date Range, Ticket Type, Customer Segment, Region, Language.

- **Tickets by Component / Service / Product**
  - *Description:* Distribution by affected component, service, or product module — identifies hotspots.
  - *Consolidation:* `COUNT(tickets) GROUP BY component` with incident-rate calculation.
  - *Filters:* Component, Service, Product, Version, Environment (Prod/Staging/Dev), Status, Date Range, Severity.

---

# 2. SLA & Response Time

*Second only to status reports — critical for customer-facing support and ITSM.*

- **SLA Compliance Rate**
  - *Description:* Percentage of tickets that met their SLA target within the period. The single most-watched KPI in customer support.
  - *Consolidation:* `(tickets_met_sla / tickets_with_sla) * 100` over date range; trended as line chart, broken down by SLA type (response/resolution).
  - *Filters:* SLA Policy, SLA Type (First Response / Next Response / Resolution), Priority, Team, Customer Segment, Service Tier, Date Range, Ticket Type, Business Hours vs 24/7.

- **SLA Breach Report**
  - *Description:* Tickets that missed their SLA target — names, durations over, owners, reasons.
  - *Consolidation:* Filter `sla_status = Breached`, join with breach duration and pause/clock-stop logs. Table + aggregation by team/agent.
  - *Filters:* SLA Type, Breach Severity (minor/major), Team, Agent, Customer, Priority, Root Cause Code, Date Range.

- **Tickets Approaching SLA Breach**
  - *Description:* Open tickets within N hours/minutes of breaching SLA — triage worklist.
  - *Consolidation:* Real-time: `sla_deadline - now() <= threshold AND status_category != Done`. Sorted by time remaining ascending.
  - *Filters:* Threshold (15m, 1h, 4h, 1d), SLA Type, Priority, Team, Assignee, Customer Tier, Escalation Status.

- **First Response Time (FRT)**
  - *Description:* Average time from ticket creation to first agent response — core responsiveness KPI.
  - *Consolidation:* `AVG(first_response_at - created_at)` excluding auto-responders; reported as median + P90 + P99 to capture distribution.
  - *Filters:* Channel, Team, Agent, Priority, Business Hours flag, Customer Segment, Date Range, Ticket Type.

- **Time to Resolution (TTR)**
  - *Description:* Average elapsed time from creation to resolution, with clock-stop periods honored.
  - *Consolidation:* `AVG(resolved_at - created_at - paused_time)` grouped by type/priority. Report mean, median, P90.
  - *Filters:* Status = Resolved/Closed, Ticket Type, Priority, Team, Agent, Customer, Date Range, Resolution Code.

- **Mean Time To Acknowledge (MTTA)**
  - *Description:* Average time between alert/ticket creation and human acknowledgement — key for on-call / incident response.
  - *Consolidation:* `AVG(acknowledged_at - created_at)` on on-call or P1/P2 tickets.
  - *Filters:* Severity, Service, On-Call Schedule, Team, Shift (Day/Night), Date Range, Escalation Level.

---

# 3. Volume & Trends

*Strategic reports — leadership and planning teams.*

- **Ticket Volume Trend**
  - *Description:* Tickets created per day/week/month — the core workload curve.
  - *Consolidation:* `COUNT(tickets) GROUP BY date_bucket(created_at)` line chart; often compared year-over-year.
  - *Filters:* Granularity (hour/day/week/month/quarter), Ticket Type, Channel, Priority, Team, Product, Compare-to-Previous-Period toggle.

- **Created vs Resolved (Flow Report)**
  - *Description:* Tickets created vs resolved per period — shows net backlog growth or reduction.
  - *Consolidation:* Two series overlaid: `COUNT(created)` and `COUNT(resolved)` per bucket; shaded area = net change.
  - *Filters:* Granularity, Ticket Type, Team, Project, Priority, Date Range, Include/Exclude Reopens.

- **Peak Hours / Ticket Heatmap**
  - *Description:* Hour-of-day × day-of-week heatmap of ticket creation for staffing decisions.
  - *Consolidation:* `COUNT(tickets) GROUP BY hour_of_day, day_of_week` on a rolling window; intensity-colored matrix.
  - *Filters:* Timezone, Channel, Ticket Type, Region, Date Range, Business Hours only, Customer Segment.

- **Top N Categories / Root Causes**
  - *Description:* The most frequent ticket categories, tags, or root causes — pareto chart.
  - *Consolidation:* `COUNT(tickets) GROUP BY category ORDER BY count DESC LIMIT N`; cumulative percentage line.
  - *Filters:* Grouping Dimension (category, tag, root cause, component), N (Top 5/10/20), Date Range, Product, Include resolved only, Exclude internal.

- **Channel Deflection & Self-Service Rate**
  - *Description:* Portion of issues resolved via self-service (KB, chatbot, portal) vs escalated to agents.
  - *Consolidation:* `deflected_count / (deflected_count + agent_tickets)` using chatbot/KB event logs joined to ticket creation.
  - *Filters:* Channel (Chatbot/KB/Community/Portal), Topic, Date Range, Language, Customer Segment, Article ID.

- **Ticket Forecast (Next N Periods)**
  - *Description:* Predictive volume forecast using time-series models (ARIMA/Prophet/LSTM).
  - *Consolidation:* Model trained on historical `created_at` series; output = point forecast + confidence bands.
  - *Filters:* Horizon (7/30/90 days), Granularity, Ticket Type, Seasonality Flag, Model (ARIMA/Prophet/LSTM), Include Holidays toggle.

---

# 4. Agent & Team Performance

*Operations and people-management reports.*

- **Agent Productivity (Tickets Closed per Agent)**
  - *Description:* Volume of tickets resolved per agent in the period — productivity baseline.
  - *Consolidation:* `COUNT(resolved) GROUP BY assignee WHERE resolved_at IN range`; normalized per working hour.
  - *Filters:* Agent, Team, Date Range, Ticket Type, Priority, Weighting Scheme (flat / by complexity / by SLA), Exclude reopens.

- **Agent Utilization**
  - *Description:* Ratio of active working time to scheduled time per agent.
  - *Consolidation:* `SUM(time_on_ticket) / SUM(scheduled_time)` from time-tracking + roster data.
  - *Filters:* Agent, Team, Shift, Date Range, Include/Exclude Breaks, Status Working vs Available.

- **Mean Time To Resolution (MTTR) by Team / Agent**
  - *Description:* Average resolution time broken down by team or agent — identifies outliers.
  - *Consolidation:* `AVG(resolved_at - created_at) GROUP BY team|agent` with median + P90.
  - *Filters:* Team, Agent, Ticket Type, Priority, Complexity, Date Range, Exclude Paused Time.

- **First Contact Resolution (FCR) Rate**
  - *Description:* Percentage of tickets resolved on first interaction — efficiency and quality indicator.
  - *Consolidation:* `COUNT(tickets WHERE reply_count = 1 AND status = Resolved) / COUNT(resolved)`.
  - *Filters:* Channel, Agent, Team, Ticket Type, Priority, Customer Segment, Date Range.

- **Workload Distribution (Equity)**
  - *Description:* Open-ticket count per agent, visualized for balance / imbalance.
  - *Consolidation:* Snapshot `COUNT(open tickets) GROUP BY assignee`; deviation from team mean computed.
  - *Filters:* Team, Role, Shift, Skills / Routing Group, Include vacation / unavailable agents.

- **Cycle Time & Lead Time**
  - *Description:* Cycle Time = start-work-to-done; Lead Time = create-to-done. Common in Agile / DevOps workflows.
  - *Consolidation:* Measured from workflow state transitions: `in_progress_at → done_at` (cycle) and `created_at → done_at` (lead). Reported as histogram + percentiles.
  - *Filters:* Workflow, Team, Ticket Type, Sprint, Date Range, Epic, Label.

- **Velocity (Tickets / Story Points per Sprint)**
  - *Description:* Team throughput per sprint — Agile capacity planning.
  - *Consolidation:* `SUM(story_points) GROUP BY sprint WHERE status = Done` across last N sprints; trend line.
  - *Filters:* Team, Board, Project, Sprint Count (last 3/6/12), Include Bugs, Point Scheme.

---

# 5. Customer & Requester

*Customer-success and account-management reports.*

- **Top Customers by Ticket Volume**
  - *Description:* Customers generating the most tickets — account-health and support-cost signal.
  - *Consolidation:* `COUNT(tickets) GROUP BY customer_id ORDER BY count DESC LIMIT N`; join with CRM for tier/revenue.
  - *Filters:* N (Top 10/25/50), Customer Tier, Industry, Region, Account Manager, Date Range, Ticket Type.

- **Tickets by Customer**
  - *Description:* Full ticket log for a selected customer/account — account-review use case.
  - *Consolidation:* Filtered list by `customer_id`; aggregates: volume, avg TTR, SLA compliance, reopen rate.
  - *Filters:* Customer, Contact, Date Range, Status, Priority, Ticket Type, Product.

- **Customer Satisfaction (CSAT)**
  - *Description:* Average CSAT score from post-resolution surveys + response distribution.
  - *Consolidation:* `AVG(csat_score) WHERE csat_score IS NOT NULL`; also `% Good` and `% Bad`. Survey response rate reported alongside.
  - *Filters:* Agent, Team, Channel, Ticket Type, Customer Segment, Language, Score Range, Date Range, Survey Version.

- **Net Promoter Score (NPS)**
  - *Description:* Promoters – Detractors (%) from NPS surveys.
  - *Consolidation:* `(%promoters - %detractors)` from survey responses; trended + drill-down to verbatims.
  - *Filters:* Segment, Region, Product, Date Range, Touchpoint (post-resolution / periodic), Customer Tier.

- **Customer Effort Score (CES)**
  - *Description:* Average effort customer reported to get their issue solved.
  - *Consolidation:* `AVG(ces_score)` from survey; cross-tabbed with channel and resolution steps.
  - *Filters:* Channel, Agent, Issue Complexity, Date Range, Customer Segment.

- **Tickets Pending Customer Response**
  - *Description:* Tickets blocked waiting on customer — clears stalls.
  - *Consolidation:* `status IN ('Waiting for Customer','Pending Reply')`; with days-waiting column.
  - *Filters:* Days Waiting Threshold, Customer, Priority, Team, Channel, Auto-close policy age.

---

# 6. Backlog & Aging

*Keeping queues healthy.*

- **Backlog by Priority**
  - *Description:* Not-yet-started tickets grouped by priority — planning view.
  - *Consolidation:* `COUNT(tickets) WHERE status IN ('Open','Backlog','To Do') GROUP BY priority`.
  - *Filters:* Priority, Team, Project, Age Bucket (<1d / 1–7d / 7–30d / 30+d), Ticket Type, Epic.

- **Aging Tickets / Stale Tickets**
  - *Description:* Open tickets that have not been updated for N days — risk of being forgotten.
  - *Consolidation:* `updated_at < now() - N days AND status_category != Done`; bucketed by age.
  - *Filters:* Age Threshold (7/14/30/60/90 days), Priority, Team, Assignee, Ticket Type, Customer Tier.

- **Work-in-Progress (WIP) by Agent**
  - *Description:* Active in-progress tickets per agent — flow limits / Kanban compliance.
  - *Consolidation:* Real-time `COUNT(tickets) WHERE status = 'In Progress' GROUP BY assignee`; WIP limit breach flag.
  - *Filters:* Team, Column/Workflow State, WIP Limit Threshold, Agent, Project.

- **Oldest Open Tickets**
  - *Description:* Tickets ordered by `created_at` ascending among open — oldest-first triage list.
  - *Consolidation:* Sorted list with age column. Often top 50.
  - *Filters:* Status, Priority, Team, Project, Customer, Age Threshold.

- **Backlog Growth / Reduction Trend**
  - *Description:* Net backlog change over time — healthy teams trend downward or stable.
  - *Consolidation:* Daily snapshot of `COUNT(open tickets)` with delta from previous day.
  - *Filters:* Granularity, Team, Project, Priority, Ticket Type, Date Range.

---

# 7. Quality & Escalation

*Process-quality reports.*

- **Reopened Tickets**
  - *Description:* Tickets that were closed and then reopened — quality indicator.
  - *Consolidation:* `COUNT(tickets WHERE status_changed FROM Closed TO Open)`; rate = reopened/closed.
  - *Filters:* Agent, Team, Ticket Type, Customer, Date Range, Reopen Count (>=N), Reason Code.

- **Escalated Tickets**
  - *Description:* Tickets escalated to higher tiers / management — severity and load indicator.
  - *Consolidation:* `COUNT(tickets WHERE escalated = true OR label = 'escalated')`; with escalation-level breakdown.
  - *Filters:* Escalation Level (T1→T2, T2→T3, Mgmt), Trigger (Manual / Auto / SLA), Team, Date Range, Customer Tier.

- **Ticket Reassignment / Bounce Rate**
  - *Description:* How often tickets change owners before resolution — routing-quality signal.
  - *Consolidation:* `AVG(count of assignee changes per ticket)`; distribution histogram.
  - *Filters:* Team, Queue, Ticket Type, Bounce Count Threshold, Date Range.

- **Rejected / Invalid / Duplicate Tickets**
  - *Description:* Volume of tickets closed as invalid, duplicate, won't-do, or cannot-reproduce.
  - *Consolidation:* `COUNT GROUP BY resolution_code` filtered to non-productive resolutions.
  - *Filters:* Resolution Code, Ticket Type, Reporter, Team, Date Range.

---

# 8. Incident Management

*ITSM-focused reports.*

- **Incident Frequency by Service**
  - *Description:* Number of incidents per affected service / component — reliability ranking.
  - *Consolidation:* `COUNT(incidents) GROUP BY service WHERE type = Incident` trended.
  - *Filters:* Service, Environment, Severity, Date Range, Business Impact, Is Major Incident flag.

- **MTTR by Severity**
  - *Description:* Mean time to restore service, segmented by severity.
  - *Consolidation:* `AVG(resolved_at - created_at) GROUP BY severity` on incident tickets; honors paused clock.
  - *Filters:* Severity (S1–S4), Service, Team, On-Call Group, Date Range, Exclude Planned Maintenance.

- **Major Incident Summary**
  - *Description:* Catalog of P1/P2 incidents with impact, duration, RCA status.
  - *Consolidation:* Filter `priority IN (Highest, High) AND type = Incident`; joined with post-mortem table.
  - *Filters:* Priority, Date Range, Service, RCA Status (Pending/Complete), Customer-impacting flag.

- **Incident Reopen Rate**
  - *Description:* Share of incidents that recur after being closed — permanent-fix signal.
  - *Consolidation:* `COUNT(incidents with reopen events) / COUNT(incidents closed)`.
  - *Filters:* Service, Severity, Team, Date Range, Reopen Window (within N days).

- **Problem Records & Recurring Incidents**
  - *Description:* Problems (ITIL) and the incident clusters they represent — root-cause management.
  - *Consolidation:* Incidents grouped by `problem_id` or clustered by similarity/signature.
  - *Filters:* Problem Status, Service, Known Error flag, Occurrence Threshold, Date Range.

---

# 9. Security & Compliance

*Regulated industries and security teams.*

- **Open Security Vulnerabilities**
  - *Description:* Active security-labeled tickets, ordered by CVSS / priority.
  - *Consolidation:* `labels = security AND status_category != Done`; joined with CVSS data.
  - *Filters:* CVSS Range, Severity, Asset / System, Team, Date Range, Discovery Source (scanner/manual/pentest).

- **Overdue Patching Tickets**
  - *Description:* Patch-management tickets past their due date.
  - *Consolidation:* `label = patching AND due_date < now() AND status != Done`.
  - *Filters:* Patch Severity, Asset Group, Environment, OS / Platform, Days Overdue.

- **Compliance Audit Findings**
  - *Description:* Tickets representing findings from internal/external audits, grouped by framework.
  - *Consolidation:* `labels IN (audit, compliance) GROUP BY framework`; status + remediation-date.
  - *Filters:* Framework (SOC2/ISO27001/PCI/GDPR/HIPAA), Finding Severity, Control ID, Auditor, Status, Remediation Due.

- **Security Incident MTTR**
  - *Description:* Mean time to contain/eradicate/recover for security incidents.
  - *Consolidation:* `AVG(phase_end - phase_start) GROUP BY phase` where phase ∈ {detect, contain, eradicate, recover}.
  - *Filters:* Phase, Severity, Attack Type (phishing/malware/insider/DDoS), Team, Date Range.

- **Access Request / Approval Report**
  - *Description:* Access-request tickets with approval latency and rejection reasons.
  - *Consolidation:* `AVG(approved_at - created_at)` and `COUNT GROUP BY decision`.
  - *Filters:* Resource Type, Approver, Decision (Approved/Rejected/Pending), Department, Date Range.

---

# 10. Financial & Business Impact

*Finance, FinOps, and exec reporting.*

- **Cost per Ticket**
  - *Description:* Fully-loaded cost to resolve the average ticket — efficiency metric.
  - *Consolidation:* `(agent_cost + tool_cost + overhead) / resolved_ticket_count` per period; segmented by channel.
  - *Filters:* Channel, Team, Ticket Type, Cost Model (fully-loaded / direct), Date Range.

- **Revenue Impact by Incident**
  - *Description:* Estimated revenue lost per incident based on affected customers × duration × ARPU.
  - *Consolidation:* `SUM(affected_customers * downtime_minutes * ARPU_per_minute)` joined from billing data.
  - *Filters:* Service, Severity, Customer Tier, Date Range, Include SLA Credits.

- **Billable Hours by Client**
  - *Description:* Time logged on tickets, broken down by billable client — for MSP / consulting.
  - *Consolidation:* `SUM(time_logged) GROUP BY client WHERE billable = true`.
  - *Filters:* Client, Contract, Billable Flag, Agent, Ticket Type, Date Range, Rate Card.

- **Resource Utilization vs Capacity**
  - *Description:* Capacity modeling — demand (incoming tickets × handle time) vs supply (staffed hours).
  - *Consolidation:* `demand_hours = count * avg_handle_time`; `utilization = demand / capacity`.
  - *Filters:* Team, Shift, Skill Group, Forecast Horizon, Include Attrition flag.

---

# 11. Advanced Analytics & Forecasting

*Mature/data-driven orgs — AI/ML-powered.*

- **Ticket Sentiment Analysis**
  - *Description:* NLP-derived sentiment (positive/neutral/negative) on ticket descriptions and customer replies.
  - *Consolidation:* NLP model (BERT/transformer) applied to text; aggregated as `% negative` trend and drill-down to flagged tickets.
  - *Filters:* Sentiment (Pos/Neu/Neg), Confidence Threshold, Channel, Product, Date Range, Language.

- **SLA Breach Risk Prediction**
  - *Description:* ML-predicted probability each open ticket will breach SLA — prioritization aid.
  - *Consolidation:* Classifier (gradient boosting / logistic regression) scored on open tickets using features (priority, age, customer tier, workload).
  - *Filters:* Risk Threshold (e.g., >70%), SLA Type, Team, Priority, Horizon (next 1h/4h/1d).

- **Auto-Categorization Accuracy**
  - *Description:* Performance of ML classifier that auto-tags incoming tickets — precision/recall per category.
  - *Consolidation:* Compare predicted vs corrected label; compute precision, recall, F1 per class.
  - *Filters:* Category, Confidence Range, Date Range, Channel, Model Version.

- **Root Cause Cluster Analysis**
  - *Description:* Unsupervised clustering of ticket descriptions to surface emerging issues not yet categorized.
  - *Consolidation:* Embedding + HDBSCAN/k-means; rendered as cluster list with representative examples and growth rate.
  - *Filters:* Cluster Size Min, Growth Rate Threshold, Date Range, Product, Language.

- **Anomaly Detection in Ticket Volume**
  - *Description:* Statistical detection of unusual spikes/drops vs seasonal baseline — early warning.
  - *Consolidation:* Prophet/STL decomposition; flag points where `|residual| > N * stddev`.
  - *Filters:* Sensitivity (N sigma), Granularity, Service, Channel, Alerting On/Off.

- **Executive Dashboard Summary**
  - *Description:* Single-page KPI roll-up for leadership: volume, SLA, CSAT, MTTR, backlog, top risks.
  - *Consolidation:* Composite of the KPIs above with traffic-light coloring vs target; one card per KPI + trend spark.
  - *Filters:* Period (Day/Week/Month/Qtr), Business Unit, Region, Product, Compare vs Prior Period, Target Override.

---

## Summary Matrix

| # | Report | Category | Market Adoption |
|---|--------|----------|-----------------|
| 1 | Tickets by Status | Operational | Universal |
| 2 | Tickets by Priority | Operational | Universal |
| 3 | Tickets by Type / Category | Operational | Universal |
| 4 | Unassigned Tickets | Operational | Very High |
| 5 | Tickets by Assignee | Operational | Very High |
| 6 | Tickets by Team / Queue | Operational | Very High |
| 7 | Tickets by Channel | Operational | Very High |
| 8 | Tickets by Component | Operational | High |
| 9 | SLA Compliance Rate | SLA | Universal (CS/ITSM) |
| 10 | SLA Breach Report | SLA | Universal (CS/ITSM) |
| 11 | Tickets Approaching SLA Breach | SLA | Very High |
| 12 | First Response Time | SLA | Universal (CS) |
| 13 | Time to Resolution | SLA | Universal |
| 14 | MTTA | SLA | High (ITSM) |
| 15 | Ticket Volume Trend | Trends | Universal |
| 16 | Created vs Resolved | Trends | Very High |
| 17 | Peak Hours Heatmap | Trends | High |
| 18 | Top N Categories | Trends | High |
| 19 | Self-Service / Deflection Rate | Trends | High |
| 20 | Ticket Forecast | Trends | Medium |
| 21 | Agent Productivity | Performance | Very High |
| 22 | Agent Utilization | Performance | High |
| 23 | MTTR by Team | Performance | Very High |
| 24 | First Contact Resolution | Performance | Very High |
| 25 | Workload Distribution | Performance | High |
| 26 | Cycle / Lead Time | Performance | High (Agile) |
| 27 | Velocity | Performance | High (Agile) |
| 28 | Top Customers by Volume | Customer | Very High |
| 29 | Tickets by Customer | Customer | Very High |
| 30 | CSAT | Customer | Universal (CS) |
| 31 | NPS | Customer | High |
| 32 | CES | Customer | Medium |
| 33 | Pending Customer Response | Customer | Very High |
| 34 | Backlog by Priority | Backlog | Universal |
| 35 | Aging Tickets | Backlog | Very High |
| 36 | WIP by Agent | Backlog | High (Kanban) |
| 37 | Oldest Open Tickets | Backlog | Very High |
| 38 | Backlog Growth Trend | Backlog | High |
| 39 | Reopened Tickets | Quality | High |
| 40 | Escalated Tickets | Quality | Very High |
| 41 | Reassignment / Bounce | Quality | Medium |
| 42 | Rejected / Duplicate | Quality | Medium |
| 43 | Incident Frequency by Service | Incident | Very High (ITSM) |
| 44 | MTTR by Severity | Incident | Very High (ITSM) |
| 45 | Major Incident Summary | Incident | High (ITSM) |
| 46 | Incident Reopen Rate | Incident | Medium |
| 47 | Problem Records | Incident | Medium (ITIL) |
| 48 | Open Security Vulnerabilities | Security | High (regulated) |
| 49 | Overdue Patching | Security | High (regulated) |
| 50 | Compliance Audit Findings | Security | High (regulated) |
| 51 | Security Incident MTTR | Security | Medium |
| 52 | Access Request Report | Security | Medium |
| 53 | Cost per Ticket | Financial | Medium |
| 54 | Revenue Impact by Incident | Financial | Medium |
| 55 | Billable Hours by Client | Financial | High (MSP) |
| 56 | Resource Utilization vs Capacity | Financial | Medium |
| 57 | Sentiment Analysis | Advanced | Growing |
| 58 | SLA Breach Risk Prediction | Advanced | Growing |
| 59 | Auto-Categorization Accuracy | Advanced | Growing |
| 60 | Root Cause Clustering | Advanced | Emerging |
| 61 | Anomaly Detection | Advanced | Emerging |
| 62 | Executive Dashboard Summary | Advanced | Very High |

**Total: 62 reports across 11 categories.**

---

## Implementation Notes

- **Data model**: All reports assume a `fact_tickets` + `fact_ticket_events` + dimension tables (users, teams, customers, products, SLA policies). See Kimball dimensional model in `22-ticket-reports-advanced.md` §2.6.
- **Refresh cadence**: Operational reports → near real-time (≤5 min). Trend / executive reports → hourly or daily batch. Advanced/ML reports → daily retraining, hourly scoring.
- **Permissions**: All reports must respect tenant/org, team, and row-level security filters.
- **Export**: Every report should support CSV/Excel export and scheduled email / Slack delivery.
- **Drill-down**: Every aggregate number should click through to the underlying ticket list with filters pre-applied.
