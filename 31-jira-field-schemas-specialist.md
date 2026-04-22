# 31 - Jira Field Schemas Specialist

## Introduction

Jira Service Management (JSM) provides a highly flexible framework for managing work items through the use of fields and their schemas. Fields are the fundamental building blocks of Jira work items, capturing essential metadata and custom information tailored to organizational needs. This document delivers an exhaustive technical exploration of Jira field schemas, field types, field configurations, and field configuration schemes. It also covers critical architectural concepts, usage patterns, and upcoming limits as of 2026, field renderers, and the distinctions between system and custom fields. This document is designed for senior Jira administrators, architects, and specialists responsible for designing scalable and maintainable Jira field schemas in large enterprise environments.

---

## 1. Jira Field Types: A Deep Dive

Jira field types can be broadly classified into predefined answer fields, free text fields, number fields, date/time fields, and read-only informational fields. Each field type serves a distinct purpose and supports different data entry methods and user interaction paradigms. Understanding these types is critical for effective field schema design.

### 1.1 Fields with Predefined Answers

Predefined answer fields restrict user input to a known set of options, enabling data consistency and improved reporting. Jira supports several such field types, many of which are especially relevant in JSM projects where asset and team data integration is common.

- **Assets Fields**: These link directly to Assets/CMDB objects, allowing users to associate requests or incidents with configuration items. This integration is crucial for IT Service Management (ITSM) processes.

- **Checkboxes**: Permit multiple selections from a predefined list of options. This field type is suitable for fields where multiple attributes must be selected, such as request categories.

- **Group Picker (single/multi-select)**: Allows selection of one or multiple groups from the Atlassian Directory. This is essential for access control or routing based on group membership.

- **Label Fields**: Labels are flexible tags that users can select from existing labels or create new ones on the fly. They are leveraged for categorization and filtering.

- **Parent Field**: Replaces legacy Parent Link and Epic Link fields, enabling hierarchical relationships between issues or work items across spaces.

- **Space Picker**: Unlike the Parent field, which links work items hierarchically, the Space Picker creates a label based on a space name, useful for cross-space categorization.

- **Radio Buttons**: Single selection from predefined options, a classic form of categorical data entry.

- **Select List (single, multi, cascading)**: The single and multi-select variants allow choosing one or many options, respectively. Cascading select lists provide hierarchical option sets where selecting a parent option filters child options, akin to folder structures.

- **Team Picker**: Enables selection of Atlassian Teams configured on the site, facilitating team-based assignment and reporting.

- **User Picker (single/multi-select)**: Allows selection of individual users, with optional restrictions based on groups or space roles to control the user pool.

- **Version Picker (single/multi-select)**: Used to specify affected or fixed versions for software-related work items.

These predefined answer fields improve report consistency and reduce human error by enforcing controlled vocabularies.

### 1.2 Date and Time Fields

Date and time fields capture temporal data and include:

- **Date Picker**: Allows users to select a calendar date. Commonly used for due dates, scheduled maintenance, or incident occurrence.

- **Date Time Picker**: Extends the Date Picker with time selection, useful for SLA tracking, incident timestamps, and audit trails.

### 1.3 Number Fields

Number fields capture numeric data as free-form text with formatting options. They support integer and decimal values within a range of -1 trillion to +1 trillion and include three formatting modes:

- **Number**: Standard numeric display with rounding to the nearest thousandth (e.g., 5.555555 → 5.556).

- **Currency**: Displays numbers with appropriate currency symbols and formatting.

- **Percentage**: Treats numbers as percentages, e.g., 0.25 displayed as 25%.

Scientific notation (e.g., 5e3 for 5000) is supported, allowing flexible data entry for large or small numbers.

Number formatting is stored globally, ensuring consistent presentation across all contexts.

### 1.4 Open Text Box Fields

There are two primary open text fields:

- **Paragraph Text Field**: Supports rich text formatting such as bold, italics, underline, and links using a wiki-style renderer. Suitable for detailed descriptions, comments, or notes.

- **Short Text Field**: Plain text only, limited to 255 characters, appropriate for concise inputs like titles or short comments.

### 1.5 Read-Only Informational Fields

Certain fields do not accept user input but provide critical metadata about work items:

- **Date of First Response**: Automatically captures the date when the first comment (excluding the reporter) was made, useful for SLA reporting.

- **Days Since Last Comment**: Shows the elapsed days since the last comment, though it is not searchable or sortable and may vary based on user permissions.

- **Participants of a Work Item**: Lists users who have contributed to the issue through comments or updates.

- **Time in Status**: Tracks how long an issue remains in specific statuses, forming the basis for average time spent in status and related charts.

---

## 2. System Fields vs. Custom Fields

Jira distinguishes between system fields and custom fields. Understanding this distinction is vital for schema design and administration.

### 2.1 System Fields

System fields are built-in Jira fields essential for core issue management and workflow. They cannot be deleted and are tightly integrated with Jira's internal processes and reports. Examples include:

| Field Name          | Description                                   | Usage Context                                   |
|---------------------|-----------------------------------------------|------------------------------------------------|
| Summary             | Title or headline of the work item            | Immutable core identifier for issues            |
| Description         | Detailed text describing the work item        | Supports rich text                              |
| Priority            | Work item urgency level                        | Influences SLA and work triage                   |
| Status              | Current state in workflow                      | Drives workflow transitions                      |
| Resolution          | Closure status                                | Marks issue completion status                    |
| Assignee            | User responsible for the issue                 | Task ownership                                   |
| Reporter            | User who created the issue                      | Accountability and communication                |
| Labels              | Tags for categorization                         | Flexible filtering and grouping                  |
| Components          | Subsystems or modules related to the issue    | Enables logical grouping                         |
| Fix Version/s       | Target release versions                         | Release management                              |
| Affects Version/s   | Versions impacted by the issue                  | Defect tracking                                 |
| Due Date            | Expected completion date                        | Scheduling and prioritization                    |
| Created, Updated    | Timestamps for creation and last update        | Audit and tracking                               |
| Resolved            | Resolution date                                 | SLA and closure metrics                          |
| Work Type (Issue Type) | Issue classification                          | Defines workflows and field schemes             |
| Space (Project)     | Project or space where the issue resides       | Organizational boundary                          |
| Attachment          | Files attached to the issue                      | Supporting evidence                              |
| Comment             | User comments                                   | Collaboration                                   |
| Linked Issues       | Relationships to other issues                   | Dependency mapping                              |
| Sprint              | Agile sprint assignment                         | Agile reporting                                 |
| Story Points        | Estimation metric                               | Agile planning                                  |
| Epic Link/Parent    | Hierarchical linkage                            | Agile and hierarchy management                   |
| Time Tracking       | Original estimate, remaining estimate, time spent | Workload tracking                             |
| Security Level      | Restricts issue visibility                       | Data sensitivity                                |
| Environment         | Technical environment information                | Contextual metadata                              |
| Votes, Watchers     | User engagement metrics                          | Prioritization and notification                  |

System fields are essential for Jira's core functionality and are subject to strict controls regarding modification or deletion.

### 2.2 Custom Fields

Custom fields are user-defined fields created to capture information not covered by system fields. They are highly flexible and can be tailored to business-specific processes. Custom fields have unique numeric IDs and type identifiers. Once created, their type cannot be changed, ensuring schema stability.

Custom fields are managed through the Jira administration interface and can be assigned to field configurations and schemes to control their behavior and visibility.

---

## 3. Creating and Managing Fields

### 3.1 Creating a New Field

Creating a custom field involves several steps to ensure it is correctly defined and integrated into the Jira schema:

First, navigate to the Jira administration console, specifically under **Settings → Work items → Fields**. Within the Fields section, select **Create new field**. You must then choose a field type from the available options; this choice is critical as the field type cannot be changed after creation. Provide a clear and descriptive name, as this name will be displayed across all instances where the field appears on work items. Optionally, add a description to assist administrators when managing fields. If the field type requires predefined options (e.g., select lists, radio buttons, checkboxes), add these options now. After finalizing these details, select **Create**.

Once created, the field must be added to a field configuration to become visible and usable on work items.

### 3.2 Editing and Deleting Fields

After creation, certain field attributes can be modified, such as the field description and options for predefined answer fields. However, the field type itself remains immutable to prevent data corruption.

Deleting a custom field is possible but must be done with caution, as it removes the field and all associated data from all work items.

---

## 4. Field Configurations and Field Configuration Schemes: Architecture and Management

The architecture of field configurations and their mapping to work types via field configuration schemes is fundamental to controlling field behavior, visibility, and validation within Jira.

### 4.1 Field Configurations

A **Field Configuration** is a collection of field definitions that dictate the behavior and presentation of fields in work items. It controls whether fields are required, optional, or hidden, and can also specify field descriptions (help text) and the renderer used for the field's display.

For example, a field configuration might mark the "Customer Impact" field as required for incident work types but optional for service requests. It can also hide fields irrelevant to certain workflows.

Field configurations are created and managed in the admin interface under **Settings → Work items → Field configurations**. Administrators can add or remove fields from the configuration, toggle the required status, and edit help text.

### 4.2 Field Configuration Schemes

A **Field Configuration Scheme** maps different field configurations to specific work types (issue types) within a space (project). This mapping enables granular control over the fields displayed for different categories of work items.

For example, in a JSM project, incidents might use a field configuration that requires "Impact" and "Urgency," while change requests use a configuration emphasizing "Change Type" and "Planned Start Date."

Each space can have only one field configuration scheme. Unmapped work types will use the default field configuration defined within the scheme.

Field configuration schemes are managed via **Settings → Work items → Field configuration schemes**, where administrators can map each work type to a field configuration.

### 4.3 Architectural Diagram of Field Configuration Relationships

```plaintext
+--------------------+       +-------------------------+       +---------------------+
|   Custom/System     |       |    Field Configuration   |       | Field Configuration  |
|       Fields        |------>|  (Defines field behavior) |------>|      Scheme          |
+--------------------+       +-------------------------+       +---------------------+
                                                                 | - Maps to Work Types |
                                                                 | - One scheme per Space|
                                                                 +---------------------+
                                                                         |
                                                                         v
                                                                 +---------------------+
                                                                 |       Space         |
                                                                 +---------------------+
```

This architecture allows for flexible, context-sensitive field behavior while maintaining centralized management.

---

## 5. Field Contexts: Restricting Field Visibility and Options

Field contexts are advanced configurations that restrict where a field appears and what options it offers. Contexts can be scoped to specific spaces or work types, enhancing schema flexibility.

Contexts provide:

- Different default values per scope

- Distinct sets of options for predefined answer fields

- User restrictions for user picker fields (restricted by group or space role)

Contexts are accessed under **Work items → Fields → More actions → Contexts and default value**.

For example, a "Priority" field might have different default values in Incident and Service Request contexts.

---

## 6. Jira 2026 Limits: Scalability Considerations

Atlassian announced new limits effective February 2026 to ensure performance and maintainability of Jira instances:

- **Field Configurations** are capped at 700 fields per configuration.

- **Field Configuration Schemes** are capped at 150 work types per scheme.

These limits are critical for architects and administrators managing large Jira instances with numerous custom fields and complex workflows. Exceeding these limits will require schema refactoring, either by consolidating fields, reducing the number of work types, or leveraging the new unified "Field schemes" experience Atlassian is introducing.

---

## 7. Field Renderers: Wiki vs. Plain Text

Field renderers determine how the field content is displayed and edited in Jira work items. The two primary renderers are:

- **Wiki Renderer**: This is the default renderer that supports rich text formatting using wiki markup or Atlassian Document Format (ADF). It enables bold, italics, underline, links, code snippets, and other text enhancements. Fields like Description or Paragraph text use this renderer to improve readability and expressiveness.

- **Plain Text Renderer**: This renderer disables formatting and displays raw text. It is appropriate for fields where formatting is not desired or could cause confusion, such as short text entries or technical identifiers.

Renderers are set per field configuration, allowing different behaviors for the same field in different contexts. This per-configuration rendering flexibility enhances customization without duplicating fields.

---

## 8. Screens and Screen Schemes: The Presentation Layer

While fields and field configurations control data capture and validation, screens control where and when fields appear during user interactions.

- **Create Screen**: Fields visible when creating a work item.

- **Edit Screen**: Fields visible when editing an existing item.

- **View Screen**: Fields visible when viewing an item.

- **Transition Screen**: Fields visible during workflow transitions.

**Screen Schemes** map these screens to operations, and **Work Type Screen Schemes** map screen schemes to work types, enabling fine-grained control over the user interface.

Fields must be added to appropriate screens to be visible. Required fields must be present on the Create screen to ensure data completeness.

---

## 9. Field Schema via Jira REST API: Data Model Overview

The Jira REST API exposes field schemas with detailed metadata, enabling automated management and integration.

A typical field schema JSON object includes:

```json
{
  "id": "customfield_10010",
  "name": "Customer Impact",
  "type": "option",             // option, string, number, date, user, group, etc.
  "custom": "com.atlassian.jira.plugin.system.customfieldtypes:select",
  "customId": 10010,
  "system": null,
  "items": "option"             // For array types, type of items in the array
}
```

System fields have system identifiers, whereas custom fields have custom type identifiers and numeric IDs.

This API-driven model supports programmatic schema management, bulk updates, and integration with external systems.

---

## 10. Transition to Unified Field Schemes (Post-2026)

Atlassian is retiring the classic Field Configurations and Field Configuration Schemes, replacing them with a unified **Field schemes** experience. This new model consolidates field behavior, visibility, and contexts into a single schema without using field contexts for visibility restrictions. This simplification aims to improve usability and performance but requires careful migration planning.

---

## 11. Practical Recommendations for Specialists

When designing Jira field schemas, specialists should consider:

- Use predefined answer fields where possible to enhance reporting consistency.

- Plan field contexts carefully to minimize complexity and avoid exceeding limits.

- Keep the total number of fields per configuration under 700 and work types per scheme under 150.

- Regularly audit unused custom fields and clean up to maintain performance.

- Leverage field configuration schemes to tailor field visibility and requirements per work type.

- Understand and configure field renderers appropriately for optimal user experience.

- Ensure all required fields are present on the Create screen.

- Prepare for the transition to unified Field schemes by monitoring Atlassian announcements.

---

## Appendix A: Summary Table of Jira Field Types

| Field Type                  | Data Type       | Input Method              | Supports Predefined Options | Renderer         | Use Case Example                     |
|-----------------------------|-----------------|---------------------------|-----------------------------|------------------|------------------------------------|
| Assets                      | Reference       | Search/Selection          | Yes                         | Wiki             | Linking CMDB items                  |
| Checkboxes                  | Multiple Choice | Multiple Selection         | Yes                         | Wiki             | Multi-attribute selection          |
| Group Picker (single/multi) | Group Reference | Dropdown/Multiple Select   | Yes                         | Wiki             | Assigning groups                   |
| Label                       | Text Tag        | Auto-complete/Free Entry   | No                          | Wiki             | Tagging and categorization         |
| Parent                      | Issue Link      | Search/Selection          | Yes                         | Wiki             | Issue hierarchy                   |
| Space Picker                | Label           | Dropdown                  | Yes                         | Wiki             | Cross-space categorization         |
| Radio Buttons               | Single Choice   | Single Selection           | Yes                         | Wiki             | Single attribute selection         |
| Select List (single/multi)  | Single/Multiple | Dropdown/Multi-select      | Yes                         | Wiki             | Categorization                    |
| Team                        | Team Reference  | Dropdown                  | Yes                         | Wiki             | Team assignment                   |
| User Picker (single/multi)  | User Reference  | Search/Multiple Select     | Yes (with restrictions)     | Wiki             | User assignment                 |
| Version Picker (single/multi)| Version Reference| Dropdown/Multi-select      | Yes                         | Wiki             | Release targeting                |
| Date Picker                 | Date            | Calendar                  | No                          | Wiki             | Due dates                        |
| Date Time Picker            | DateTime        | Calendar + Time           | No                          | Wiki             | Timestamping                    |
| Number Field                | Number          | Free text (number only)   | No                          | Wiki             | Metrics, estimations              |
| Paragraph Text              | Text            | Multi-line Text           | No                          | Wiki             | Descriptions, detailed notes     |
| Short Text                  | Text            | Single line Text          | No                          | Plain Text       | Short identifiers                |
| Read-Only Fields            | Various         | N/A                       | N/A                         | Wiki or Plain    | Metadata display                 |

---

## Appendix B: Detailed Field Configuration Example

Consider a JSM project with three work types: Incident, Service Request, and Change. The administrator creates two field configurations: one for Incident and one for Service Request and Change.

The Incident configuration marks the "Impact" and "Urgency" fields as required and visible, with the Description field using the Wiki renderer. The Service Request configuration hides "Impact" and "Urgency" fields and marks "Requested Service" as required.

These configurations are mapped in a Field Configuration Scheme:

| Work Type        | Field Configuration       |
|------------------|---------------------------|
| Incident         | Incident Field Configuration|
| Service Request  | Service Request Config     |
| Change           | Service Request Config     |

This setup ensures each work type prompts users for relevant information while maintaining a consistent schema.

---

## Appendix C: Field Configuration Limits and Schema Size Impact

The 2026 field limits are enforced to maintain Jira's operational performance and usability. Exceeding 700 fields in a configuration or 150 work types in a scheme can lead to performance degradation.

Large organizations with diverse processes must adopt best practices including:

- Consolidating similar fields.

- Using field contexts sparingly.

- Archiving or removing obsolete fields.

- Utilizing the new unified Field schemes when available.

---

## Conclusion

Mastering Jira field schemas requires a comprehensive understanding of field types, system versus custom fields, field configurations, and field configuration schemes. With the impending 2026 limits and schema simplifications, specialists must architect resilient, scalable, and maintainable field schemas tailored to organizational workflows. Combining this deep technical knowledge with practical administration skills ensures Jira Service Management instances are optimized for efficiency, usability, and scalability.

---

*This document is based entirely on Atlassian's official documentation and community knowledge as of 2024.*