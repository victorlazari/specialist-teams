# Jira Status & Workflows: An In-Depth Specialist Guide

---

## Introduction

Jira, developed by Atlassian, has become a cornerstone tool for issue tracking, project management, and IT service management (ITSM). Central to Jira's flexibility and power are its status definitions, request types, workflows, and their interrelations. This document delves deeply into the mechanisms of **Jira statuses**, **request types vs work types**, **workflow transitions**, and the architecture of **ITSM workflows** in Jira Service Management (JSM), as prescribed by Atlassian’s official documentation and best practices.

The goal is to provide a comprehensive understanding of how Jira’s statuses and workflows operate, impact reporting and visualization, and how administrators can leverage these elements to build robust ITSM processes with optimal clarity and control.

---

## 1. Request Types vs Work Types (Issue Types) in Jira Service Management

In Jira Service Management, the concepts of **request types** and **work types** (typically represented as *issue types* in Jira Software and Service Management) are foundational yet distinct elements that govern how customers interact with the service portal and how agents process work internally.

### 1.1. Request Types

Request types are designed primarily from the *customer perspective*. They define the types of requests that end users can submit through the customer portal. Each request type corresponds to one or more issue types but serves as a simplified interface to categorize and tailor customer interactions.

Request types are characterized by the following:

- **Customer-facing labels and descriptions**: These simplify the selection process for customers, abstracting away internal complexities.
- **Forms and fields**: Request types determine which fields are shown on the portal and can be customized independently of the underlying issue type.
- **Workflows and SLAs**: While request types don’t have workflows themselves, they route to issue types which follow workflows with SLA policies attached.

For example, a customer-facing request type might be “Password Reset” which maps internally to the issue type “Service Request”.

### 1.2. Work Types (Issue Types)

Work types, or issue types, represent the *internal categorization of work*. They define the nature of the item being worked on, and each issue type can have its own:

- **Workflow**: Different workflows can be assigned to different issue types.
- **Fields and screens**: The data collected and displayed can vary by issue type.
- **Permissions and automation rules**: These can be configured based on issue type.

In Jira Service Management, common issue types include **Service Request**, **Incident**, **Problem**, **Change**, and **Task**. Each corresponds to different types of work with specific workflows reflective of ITIL best practices.

### 1.3. Relationship Between Request Types and Issue Types

Request types map to one or more issue types but are not interchangeable. This relationship allows administrators to tailor the customer experience independently of internal processes. For example, multiple request types such as “New Laptop Request” and “Software Installation Request” may map to the same issue type “Service Request” but differ in the fields and forms shown to the customer.

| Aspect              | Request Types                        | Work Types (Issue Types)              |
|---------------------|------------------------------------|-------------------------------------|
| Audience            | Customers/End Users                 | Agents/Service Teams                 |
| Purpose             | Simplify request submission        | Categorize and route work internally|
| Customization       | Portal forms, customer view        | Workflows, fields, screens          |
| Mapping             | One or many to issue types         | Core Jira entity for work tracking  |
| Visibility          | Customer portal                     | Jira issue navigator and boards     |

---

## 2. The Four ITSM Work Categories in Jira Service Management

Jira Service Management is designed to support IT Service Management (ITSM) practices and frameworks such as ITIL. Atlassian categorizes ITSM work into four core categories, each with distinct workflows, SLAs, and issue types.

### 2.1. Service Requests

Service Requests are formal requests from users for information, advice, standard changes, or access to a service. These are typically routine, low-risk requests that follow a predefined workflow.

Examples include requests for password resets, access to software, or equipment provisioning. Service Requests usually map to the “Service Request” issue type.

### 2.2. Incidents

Incidents represent unplanned interruptions or degradations of service, requiring immediate attention to restore normal service operation.

Incident management workflows emphasize rapid triage, diagnosis, and resolution, often involving urgent escalation paths. Incidents typically use the “Incident” issue type.

### 2.3. Problems

Problems are the underlying causes of one or more incidents. Problem management focuses on root cause analysis and long-term fixes, following a more investigative and analytical workflow.

Problems usually map to the “Problem” issue type and may be linked to multiple incident tickets.

### 2.4. Changes

Changes refer to the addition, modification, or removal of anything that could impact IT services. Change management workflows are designed to minimize risk and ensure proper authorization and documentation.

Change requests are handled via the “Change” issue type, often incorporating approval steps, scheduled implementations, and rollback procedures.

| ITSM Category    | Description                                            | Typical Issue Type    | Workflow Focus                         |
|-----------------|--------------------------------------------------------|----------------------|--------------------------------------|
| Service Requests | User requests for standard services or information     | Service Request      | Efficient fulfillment with minimal risk |
| Incidents       | Unplanned service interruptions requiring fast resolution | Incident             | Rapid triage, restoration             |
| Problems        | Root cause analysis of incidents                        | Problem              | Investigation, long-term resolution   |
| Changes         | Controlled modifications to IT services                 | Change               | Risk assessment, approval, implementation |

---

## 3. Jira Status Categories and Their Impact on Reporting and Visualization

Statuses in Jira represent the current state of an issue within its workflow. However, for reporting, dashboards, and visualizations, Jira groups statuses into broader **status categories**: **To Do**, **In Progress**, and **Done**.

### 3.1. Status Categories Explained

- **To Do**: This category includes all statuses that represent work that has not yet been started. Common examples are “Open”, “Backlog”, or “Waiting”.
  
- **In Progress**: Statuses that indicate active work on the issue, such as “In Progress”, “Under Review”, or “Testing”.

- **Done**: Statuses that denote completion or closure of the issue, for example, “Resolved”, “Closed”, or “Completed”.

Every Jira status must be assigned to exactly one status category. This grouping is crucial for simplifying reporting and visual cues.

### 3.2. Impact on Reporting and Visualization

Jira leverages status categories extensively in:

- **Kanban and Scrum boards**: Columns typically represent status categories, providing a high-level view of workflow progress.
- **Reports and Gadgets**: Many built-in reports, such as control charts, cumulative flow diagrams, and sprint reports, are based on status category groupings.
- **SLAs and Automation**: SLA timers often start or stop depending on the transition into or out of a specific status category.

For example, an SLA might measure time elapsed while an issue is in the “In Progress” category, ignoring time spent in “To Do” or “Done”.

### 3.3. Custom Statuses and Categories

While Jira allows custom statuses, it is critical to assign them correctly to one of the three status categories to maintain accurate reporting and data consistency. Misclassification can lead to misleading reports and broken automation.

| Status Category | Description                              | Examples of Statuses             | Reporting Impact                               |
|-----------------|------------------------------------------|--------------------------------|------------------------------------------------|
| To Do           | Work not started                         | Open, Backlog, Waiting          | Issues appear as pending or backlog in reports |
| In Progress     | Work actively being performed            | In Progress, Under Review       | Issues counted as active work in metrics        |
| Done            | Work completed or issue closed           | Resolved, Closed, Completed     | Issues counted as completed, stop SLA timers    |

---

## 4. Workflow Fundamentals and the Workflow Editor

Workflows define the lifecycle of an issue from creation to completion. They consist of statuses and transitions that dictate how issues move through different states.

### 4.1. Core Concepts of Workflows

A **workflow** is a directed graph composed of nodes (statuses) and edges (transitions). It controls:

- The possible states an issue can be in.
- The valid paths for moving between states.
- The business logic governing transitions.

Every Jira issue type is associated with a workflow, which can be customized to fit organizational processes.

### 4.2. Workflow Components

- **Statuses**: Represent the state of an issue (e.g., Open, In Progress, Closed).
- **Transitions**: Actions that move an issue from one status to another (e.g., “Start Progress”, “Resolve Issue”).
- **Conditions, validators, triggers, post-functions**: Additional rules and automation applied during transitions (discussed in detail below).

### 4.3. The Jira Workflow Editor

Atlassian provides a graphical workflow editor within the Jira administration interface. This editor allows users to:

- Add, remove, or rename statuses.
- Draw transitions between statuses.
- Configure transition properties including conditions, validators, triggers, and post-functions.
- Set the initial status of the workflow.

The editor displays the workflow as a flowchart, simplifying visualization of complex processes.

### 4.4. Workflow Schemes

Workflows are linked to issue types through **workflow schemes**, which define which workflow applies to which issue type within a project. This modular approach allows different issue types to follow different lifecycles.

---

## 5. Detailed Breakdown of Transition Rules: Triggers, Conditions, Validators, and Post-Functions

Transitions are the dynamic element of workflows allowing issues to move between statuses. Each transition can be customized using four types of rules:

### 5.1. Triggers

Triggers automate transitions in response to events, often from linked development tools or other Jira entities. For example, a trigger can move an issue to “In Progress” when a linked Git branch is created or when a Bitbucket pull request is opened.

Triggers are essential for integrating Jira with external devops tools, enabling continuous delivery pipelines and reducing manual updates.

### 5.2. Conditions

Conditions control whether a transition should be available to a user. If a condition is not met, the transition button does not appear on the issue screen for the user.

Common conditions include:

- Checking user permissions (e.g., only users with “Developer” role can transition).
- Ensuring the issue is assigned.
- Restricting transitions to issues in specific projects or with certain field values.

Conditions are critical for enforcing business rules and security.

### 5.3. Validators

Validators run when a transition is attempted, verifying that the data submitted meets required criteria before allowing the transition to proceed.

Examples of validators include:

- Ensuring required fields are filled.
- Validating the format of a field (e.g., date fields).
- Checking that the user has permission to edit the issue.

If a validator fails, the transition is blocked, and the user receives an error message explaining the failure.

### 5.4. Post-Functions

Post-functions execute after a transition completes successfully. They automate follow-up actions such as:

- Updating fields (e.g., setting Resolution).
- Generating comments.
- Sending notifications.
- Creating linked issues or updating linked tickets.
- Re-indexing the issue.

Post-functions are vital for maintaining data integrity and automating routine steps.

---

## 6. The Resolution Field vs the Status Field: Open vs Closed States

A common point of confusion in Jira is the difference between the **Status** field and the **Resolution** field, especially in terms of defining when an issue is considered “closed” or “done.”

### 6.1. Status Field

The Status field describes the current position of the issue within its workflow. It is a visual and functional indicator of progress, such as “Open,” “In Progress,” or “Resolved.”

Statuses are grouped into categories (To Do, In Progress, Done) that influence reporting and board visualizations.

### 6.2. Resolution Field

The Resolution field indicates the outcome of an issue. Unlike Status, it is a field that typically remains empty (null) while an issue is active and is set only when the issue is resolved or closed.

Common resolution values include “Fixed,” “Won’t Fix,” “Duplicate,” and “Incomplete.”

### 6.3. Relationship and Best Practices

- An issue is considered **open** if the Resolution field is empty, regardless of status.
- An issue is considered **closed** if the Resolution field is set to a value (non-empty).

This distinction is critical because many Jira filters, reports, and SLAs use Resolution to determine whether an issue should be included or excluded.

For example, a query like `resolution = Unresolved` returns all issues where Resolution is empty (i.e., open issues), even if the status is “Resolved” but Resolution was never set.

### 6.4. Workflow Considerations for Resolution

Typically, the Resolution field is set in a **post-function** on a transition into a “Done” category status. This ensures that when an issue is moved to a closed status, its Resolution is properly updated.

It is important to have workflows configured so that:

- Resolution is cleared when reopening an issue.
- Resolution is set only upon closure.
- Transitions to “Done” statuses enforce Resolution setting via post-functions.

---

## 7. Structuring Portal Groups and Queues for Request Types

Effective organization of the customer portal and agent queues is essential for efficient service delivery and user experience.

### 7.1. Portal Groups in Jira Service Management

Portal groups organize request types into meaningful categories on the customer portal. These categories help users quickly locate the relevant request type.

For example, portal groups may be organized into “IT Support,” “HR Requests,” and “Facilities.” Each group contains related request types such as “Password Reset” under IT Support.

Portal groups are configured by Jira administrators and can be customized in terms of:

- Group name
- Display order
- Request types included

This organization enhances usability and optimizes request routing.

### 7.2. Queues for Request Types

Queues are agent-side filters that organize incoming requests for processing. Queues are typically based on criteria such as:

- Request type
- Status
- Priority
- SLA breaches

For example, a queue might display all open Incidents assigned to a specific team or all Service Requests waiting for approval.

Agents can use queues to prioritize work, monitor SLA compliance, and collaborate effectively.

### 7.3. Best Practices in Structuring Portals and Queues

An effective portal design aligns request types with customer needs and business functions, minimizing confusion and redundant requests.

Similarly, queues should be designed to reflect operational priorities, such as separating urgent Incidents from routine Service Requests, enabling agents to focus on critical work first.

| Element        | Purpose                                     | Configuration Example                          | Impact                                    |
|----------------|---------------------------------------------|-----------------------------------------------|-------------------------------------------|
| Portal Groups  | Categorize request types for customers      | “IT Support” containing “Password Reset”      | Improved user navigation and request accuracy |
| Request Types  | Define specific request categories          | “New Laptop Request”, “Software Installation”| Tailored forms, workflows, and SLAs        |
| Queues         | Organize agent work by request attributes   | “Open Incidents”, “Requests Awaiting Approval”| Efficient workload management and SLA monitoring |

---

## Conclusion

Jira’s status and workflow architecture, combined with request types and issue types, provide a powerful framework for managing ITSM processes and general work management. A deep understanding of these components, their relationships, and configurations is essential for administrators and service managers aiming to optimize Jira Service Management for their organizations.

Mastering the configuration of request types vis-à-vis work types, aligning statuses with correct status categories, designing workflows with precise transitions, and structuring portals and queues effectively creates a seamless experience for both customers and agents. This ensures high productivity, clear visibility, and adherence to ITIL best practices.

---

## References

1. Atlassian Documentation: [Jira Service Management Request Types](https://support.atlassian.com/jira-service-management/docs/configure-request-types/)
2. Atlassian Documentation: [Issue Types in Jira Service Management](https://support.atlassian.com/jira-service-management-cloud/docs/about-issue-types/)
3. Atlassian Documentation: [ITSM Practices in Jira Service Management](https://www.atlassian.com/itsm)
4. Atlassian Documentation: [Jira Workflows](https://support.atlassian.com/jira-cloud-administration/docs/manage-workflows/)
5. Atlassian Documentation: [Workflow Transitions and Post-functions](https://support.atlassian.com/jira-cloud-administration/docs/workflow-transitions/)
6. Atlassian Documentation: [Status Categories](https://support.atlassian.com/jira-cloud-administration/docs/status-categories/)
7. Atlassian Documentation: [Resolution Field in Jira](https://support.atlassian.com/jira-cloud-administration/docs/configure-issue-resolution/)
8. Atlassian Documentation: [Configure Queues in Jira Service Management](https://support.atlassian.com/jira-service-management-cloud/docs/configure-queues/)
9. Atlassian Documentation: [Customize the Customer Portal](https://support.atlassian.com/jira-service-management-cloud/docs/customize-your-portal/)

---

*End of Document*