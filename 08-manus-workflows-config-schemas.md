# manus-workflows Configuration Schemas Guide

## 1. Introduction

Welcome to the comprehensive Configuration Schemas guide for **manus-workflows**. This document serves as the definitive reference for configuring, tuning, and managing the behavior of the manus-workflows engine. Whether you are deploying a simple automation script or orchestrating a complex, multi-stage enterprise pipeline, understanding the configuration schemas is paramount to ensuring reliability, performance, and security.

In modern distributed systems, configuration is as critical as code. The manus-workflows engine relies on a set of meticulously designed YAML and JSON schemas to define workflow topologies, execution parameters, resource allocations, and integration endpoints. This guide will walk you through every configuration file, detailing each field, its default value, accepted data types, and best practices for production environments.

The configuration ecosystem of manus-workflows is divided into several key areas:
- **Global Engine Configuration (`manus-global.yaml`)**: Dictates the core behavior of the workflow engine, including thread pools, memory limits, and logging verbosity.
- **Workflow Definition Schema (`workflow-schema.json`)**: The structural blueprint for defining individual workflows, tasks, dependencies, and retry logic.
- **Integration & Connector Configuration (`connectors.yaml`)**: Manages credentials, endpoints, and timeouts for external services.
- **Security & Access Control (`rbac-config.yaml`)**: Defines roles, permissions, and authentication mechanisms.

By mastering these schemas, administrators and developers can unlock the full potential of manus-workflows, ensuring that pipelines execute efficiently and securely under varying loads.

## 2. Global Engine Configuration (`manus-global.yaml`)

The `manus-global.yaml` file is the heart of the manus-workflows engine. It is typically located in `/etc/manus/` or specified via the `MANUS_CONFIG_PATH` environment variable. This file controls the foundational parameters that dictate how the engine operates on a host or within a containerized environment.

### 2.1 Core Settings

The `core` section defines the fundamental operational parameters of the engine.

- **`node_id`** (String)
  - **Description**: A unique identifier for the engine instance in a clustered environment.
  - **Default**: Auto-generated UUID.
  - **Best Practice**: Set this to the hostname or pod name in Kubernetes environments to facilitate easier debugging and log tracing.

- **`environment`** (String)
  - **Description**: The operational environment (e.g., `development`, `staging`, `production`).
  - **Default**: `development`
  - **Best Practice**: Always explicitly set this to `production` in live environments to enable stricter security checks and optimized performance profiles.

- **`data_dir`** (String)
  - **Description**: The absolute path to the directory where the engine stores temporary state, local caches, and execution logs.
  - **Default**: `/var/lib/manus/data`
  - **Best Practice**: Ensure this directory is mounted on a high-performance SSD, especially for I/O-intensive workflows.

### 2.2 Execution Engine

The `execution` section governs how workflows are scheduled and executed.

- **`max_concurrent_workflows`** (Integer)
  - **Description**: The maximum number of workflows that can run simultaneously on this node.
  - **Default**: `100`
  - **Best Practice**: Tune this based on available CPU and memory. For CPU-bound tasks, set this close to the number of logical cores. For I/O-bound tasks, a higher number may be appropriate.

- **`worker_thread_pool_size`** (Integer)
  - **Description**: The number of threads allocated for executing individual tasks within workflows.
  - **Default**: `500`
  - **Best Practice**: Monitor thread contention and adjust accordingly. Too many threads can lead to excessive context switching overhead.

- **`default_timeout_seconds`** (Integer)
  - **Description**: The default maximum execution time for a workflow if not explicitly defined in the workflow schema.
  - **Default**: `3600` (1 hour)
  - **Best Practice**: Keep this as low as reasonably possible to prevent runaway workflows from consuming resources indefinitely.

### 2.3 Database and State Management

manus-workflows requires a persistent store to track workflow state, history, and metadata.

- **`database.type`** (String)
  - **Description**: The type of database backend. Supported values: `postgres`, `mysql`, `sqlite`.
  - **Default**: `sqlite`
  - **Best Practice**: Use `postgres` for production environments due to its robust concurrency control and JSON support.

- **`database.connection_string`** (String)
  - **Description**: The URI for connecting to the database.
  - **Default**: `sqlite:////var/lib/manus/data/state.db`
  - **Best Practice**: Never hardcode credentials in this string. Use environment variable interpolation (e.g., `${DB_CONNECTION_STRING}`).

- **`database.pool_size`** (Integer)
  - **Description**: The maximum number of database connections in the pool.
  - **Default**: `20`
  - **Best Practice**: Align this with the `worker_thread_pool_size` and the database server's maximum connection limits.

### 2.4 Logging and Telemetry

Proper observability is crucial for maintaining healthy workflows.

- **`logging.level`** (String)
  - **Description**: The verbosity of the logs. Supported values: `DEBUG`, `INFO`, `WARN`, `ERROR`.
  - **Default**: `INFO`
  - **Best Practice**: Use `INFO` for production and `DEBUG` only during active troubleshooting to avoid excessive disk I/O and storage costs.

- **`logging.format`** (String)
  - **Description**: The format of the log output. Supported values: `text`, `json`.
  - **Default**: `text`
  - **Best Practice**: Use `json` in production to facilitate easy parsing by centralized logging systems like ELK or Splunk.

- **`telemetry.enabled`** (Boolean)
  - **Description**: Whether to export metrics to a telemetry backend (e.g., Prometheus).
  - **Default**: `false`
  - **Best Practice**: Always enable this in production to monitor engine health, workflow success rates, and latency.

## 3. Workflow Definition Schema (`workflow-schema.json`)

The workflow definition schema dictates how individual workflows are structured. This schema is typically authored in YAML or JSON and validated against a strict JSON Schema before execution.

### 3.1 Metadata

Every workflow must begin with metadata that identifies and describes it.

- **`version`** (String)
  - **Description**: The schema version used by the workflow.
  - **Required**: Yes
  - **Example**: `"1.0"`

- **`name`** (String)
  - **Description**: A unique, human-readable name for the workflow.
  - **Required**: Yes
  - **Example**: `"daily-etl-pipeline"`

- **`description`** (String)
  - **Description**: A detailed explanation of what the workflow accomplishes.
  - **Required**: No
  - **Best Practice**: Always provide a clear description to aid future maintainers.

### 3.2 Triggers

Triggers define how and when a workflow is initiated.

- **`triggers`** (Array of Objects)
  - **Description**: A list of events that can start the workflow.
  - **Supported Types**: `cron`, `webhook`, `event`.

#### Cron Trigger
- **`type`**: `"cron"`
- **`schedule`**: A standard cron expression (e.g., `"0 0 * * *"` for daily at midnight).
- **`timezone`**: The timezone for the schedule (e.g., `"UTC"`).

#### Webhook Trigger
- **`type`**: `"webhook"`
- **`endpoint`**: The specific path to listen on (e.g., `"/webhooks/github"`).
- **`authentication`**: The required authentication method (e.g., `"hmac"`, `"bearer"`).

### 3.3 Tasks

Tasks are the fundamental building blocks of a workflow. They represent individual units of work.

- **`tasks`** (Object)
  - **Description**: A dictionary of task definitions, keyed by a unique task ID.

#### Task Properties

- **`type`** (String)
  - **Description**: The type of action the task performs (e.g., `http_request`, `execute_script`, `run_container`).
  - **Required**: Yes

- **`depends_on`** (Array of Strings)
  - **Description**: A list of task IDs that must complete successfully before this task can start.
  - **Required**: No
  - **Best Practice**: Use this to define the Directed Acyclic Graph (DAG) of your workflow.

- **`inputs`** (Object)
  - **Description**: The parameters passed to the task. The structure depends on the task `type`.
  - **Required**: Yes

- **`retries`** (Object)
  - **Description**: Configuration for handling transient failures.
  - **Properties**:
    - `max_attempts` (Integer): Maximum number of retries. Default: `0`.
    - `delay_seconds` (Integer): Base delay between retries. Default: `5`.
    - `backoff_multiplier` (Float): Multiplier for exponential backoff. Default: `1.0`.

- **`timeout_seconds`** (Integer)
  - **Description**: The maximum time allowed for this specific task to complete.
  - **Required**: No
  - **Best Practice**: Always set a timeout for tasks that interact with external systems to prevent indefinite hangs.

### 3.4 Example Workflow Definition

```yaml
version: "1.0"
name: "user-onboarding"
description: "Processes new user registrations and provisions resources."
triggers:
  - type: "webhook"
    endpoint: "/api/v1/users/register"
tasks:
  validate_input:
    type: "execute_script"
    inputs:
      script: "validate.py"
      args: ["{{ trigger.payload }}"]
  create_database_record:
    type: "http_request"
    depends_on: ["validate_input"]
    inputs:
      method: "POST"
      url: "http://internal-db-api/users"
      body: "{{ tasks.validate_input.output }}"
    retries:
      max_attempts: 3
      delay_seconds: 2
      backoff_multiplier: 2.0
  send_welcome_email:
    type: "http_request"
    depends_on: ["create_database_record"]
    inputs:
      method: "POST"
      url: "https://api.sendgrid.com/v3/mail/send"
      headers:
        Authorization: "Bearer {{ secrets.SENDGRID_API_KEY }}"
```

## 4. Integration & Connector Configuration (`connectors.yaml`)

Workflows rarely exist in isolation. They frequently interact with external APIs, databases, and message queues. The `connectors.yaml` file centralizes the configuration for these external integrations, promoting reuse and secure credential management.

### 4.1 Connector Definition

Each connector is defined under the `connectors` key.

- **`name`** (String)
  - **Description**: The unique identifier for the connector, used in workflow definitions.

- **`type`** (String)
  - **Description**: The protocol or service type (e.g., `rest`, `graphql`, `aws_s3`, `kafka`).

- **`configuration`** (Object)
  - **Description**: The specific settings required for the connector type.

### 4.2 REST Connector Configuration

- **`base_url`** (String)
  - **Description**: The root URL for the API.
  - **Required**: Yes

- **`authentication`** (Object)
  - **Description**: How to authenticate with the API.
  - **Supported Types**: `basic`, `bearer`, `oauth2`, `custom_header`.

- **`timeout_seconds`** (Integer)
  - **Description**: The default timeout for requests using this connector.
  - **Default**: `30`

### 4.3 AWS S3 Connector Configuration

- **`region`** (String)
  - **Description**: The AWS region.
  - **Required**: Yes

- **`bucket`** (String)
  - **Description**: The default bucket to interact with.
  - **Required**: No

- **`credentials`** (Object)
  - **Description**: AWS credentials.
  - **Best Practice**: Prefer using IAM roles (e.g., `use_iam_role: true`) over hardcoding `access_key_id` and `secret_access_key`.

## 5. Security & Access Control (`rbac-config.yaml`)

Security is a paramount concern, especially when workflows have the power to modify infrastructure or access sensitive data. manus-workflows implements a robust Role-Based Access Control (RBAC) system configured via `rbac-config.yaml`.

### 5.1 Roles

Roles define a set of permissions.

- **`roles`** (Array of Objects)
  - **Description**: A list of role definitions.

#### Role Properties

- **`name`** (String)
  - **Description**: The name of the role (e.g., `admin`, `developer`, `viewer`).

- **`permissions`** (Array of Strings)
  - **Description**: The specific actions allowed by this role.
  - **Format**: `resource:action` (e.g., `workflow:read`, `workflow:execute`, `secrets:write`).

### 5.2 Bindings

Bindings associate users or groups with specific roles.

- **`bindings`** (Array of Objects)
  - **Description**: A list of role bindings.

#### Binding Properties

- **`role`** (String)
  - **Description**: The name of the role being assigned.

- **`subjects`** (Array of Objects)
  - **Description**: The users or groups receiving the role.
  - **Properties**:
    - `kind` (String): `User` or `Group`.
    - `name` (String): The identifier of the user or group.

### 5.3 Example RBAC Configuration

```yaml
roles:
  - name: "workflow-developer"
    permissions:
      - "workflow:read"
      - "workflow:create"
      - "workflow:update"
      - "execution:read"
      - "execution:start"
      - "execution:stop"
  - name: "auditor"
    permissions:
      - "workflow:read"
      - "execution:read"
      - "logs:read"

bindings:
  - role: "workflow-developer"
    subjects:
      - kind: "Group"
        name: "engineering-team"
  - role: "auditor"
    subjects:
      - kind: "User"
        name: "compliance-officer@company.com"
```

## 6. Advanced Configuration and Tuning

For enterprise deployments, standard configurations may not suffice. This section covers advanced tuning parameters.

### 6.1 Memory Management

manus-workflows is designed to be memory efficient, but large payloads can cause spikes.

- **`memory.max_payload_size_mb`** (Integer)
  - **Description**: The maximum size of data that can be passed between tasks.
  - **Default**: `10`
  - **Best Practice**: If workflows process large files, do not increase this limit. Instead, pass references (e.g., S3 URIs) between tasks and stream the data directly to disk or the destination service.

- **`memory.garbage_collection_interval_seconds`** (Integer)
  - **Description**: How frequently the engine forces garbage collection of completed workflow states from memory.
  - **Default**: `300`

### 6.2 High Availability and Clustering

To ensure fault tolerance, manus-workflows can be deployed in a cluster.

- **`cluster.enabled`** (Boolean)
  - **Description**: Enables clustering mode.
  - **Default**: `false`

- **`cluster.discovery_mechanism`** (String)
  - **Description**: How nodes find each other. Supported values: `static`, `consul`, `kubernetes`.
  - **Default**: `static`

- **`cluster.heartbeat_interval_seconds`** (Integer)
  - **Description**: How often nodes send heartbeats to the cluster manager.
  - **Default**: `5`

- **`cluster.node_timeout_seconds`** (Integer)
  - **Description**: The time after which a node is considered dead if no heartbeat is received.
  - **Default**: `15`
  - **Best Practice**: Ensure this is at least 3 times the `heartbeat_interval_seconds` to prevent false positives during temporary network blips.

## 7. Configuration Validation and Deployment

Before deploying configuration changes to a production environment, it is critical to validate them.

### 7.1 CLI Validation

The manus-workflows CLI provides a command to validate configuration files against their respective schemas.

```bash
manus-cli config validate --file /etc/manus/manus-global.yaml
manus-cli workflow validate --file my-workflow.yaml
```

### 7.2 CI/CD Integration

It is highly recommended to store all configuration files in version control (e.g., Git) and use a CI/CD pipeline to deploy them.

1. **Linting**: Run YAML/JSON linters to catch syntax errors.
2. **Validation**: Use the `manus-cli` to validate the schemas.
3. **Testing**: Deploy the configuration to a staging environment and run a suite of integration tests.
4. **Deployment**: Use tools like Ansible, Terraform, or Helm to apply the configuration to the production environment.

## 8. Troubleshooting Configuration Issues

When things go wrong, the configuration is often the first place to look.

### 8.1 Common Errors

- **`SchemaValidationError`**: This indicates that a configuration file does not conform to the expected schema. Check the error message for the specific line number and field that failed validation.
- **`ConnectionRefused`**: Often caused by incorrect database connection strings or connector endpoints. Verify the URLs and ensure network firewalls allow the traffic.
- **`AuthenticationFailed`**: Check the credentials in `connectors.yaml` or the database connection string. Ensure secrets are being correctly interpolated from environment variables.

### 8.2 Debugging Techniques

- **Enable Debug Logging**: Temporarily set `logging.level` to `DEBUG` in `manus-global.yaml` to get more detailed information about how the engine is parsing the configuration.
- **Inspect Environment Variables**: If you are using environment variable interpolation, ensure the variables are actually set in the environment where the engine is running.
- **Use the CLI**: The `manus-cli config dump` command can be used to view the final, resolved configuration after all defaults and environment variables have been applied.

## 9. Conclusion

A deep understanding of the manus-workflows configuration schemas is essential for building robust, scalable, and secure automation pipelines. By carefully tuning the global engine parameters, structuring workflows efficiently, managing integrations securely, and implementing strict access controls, you can ensure that your manus-workflows deployment meets the demands of your enterprise.

Always remember to treat configuration as code: version it, test it, and deploy it automatically. With these best practices in place, manus-workflows will serve as a reliable foundation for your organization's automation needs.


## 10. Deep Dive: Advanced Connector Configurations

To further expand on the capabilities of manus-workflows, let's explore some of the more advanced connector configurations available for enterprise use cases.

### 10.1 Kafka Connector

For event-driven architectures, the Kafka connector is indispensable.

- **`bootstrap_servers`** (Array of Strings)
  - **Description**: A list of host:port pairs used for establishing the initial connection to the Kafka cluster.
  - **Required**: Yes

- **`security_protocol`** (String)
  - **Description**: Protocol used to communicate with brokers. Supported values: `PLAINTEXT`, `SSL`, `SASL_PLAINTEXT`, `SASL_SSL`.
  - **Default**: `PLAINTEXT`

- **`sasl_mechanism`** (String)
  - **Description**: SASL mechanism used for authentication. Supported values: `PLAIN`, `GSSAPI`, `OAUTHBEARER`, `SCRAM-SHA-256`, `SCRAM-SHA-512`.
  - **Required**: If `security_protocol` is `SASL_PLAINTEXT` or `SASL_SSL`.

- **`consumer_group_id`** (String)
  - **Description**: A unique string that identifies the consumer group this consumer belongs to.
  - **Required**: Yes, for trigger configurations.

### 10.2 GraphQL Connector

Interacting with modern APIs often requires GraphQL support.

- **`endpoint`** (String)
  - **Description**: The URL of the GraphQL endpoint.
  - **Required**: Yes

- **`default_headers`** (Object)
  - **Description**: Headers to include in every request, such as authentication tokens.
  - **Required**: No

- **`introspection_enabled`** (Boolean)
  - **Description**: Whether the connector should attempt to fetch the schema via introspection for validation purposes.
  - **Default**: `true`

## 11. Best Practices for Secret Management

Handling sensitive information like API keys, database passwords, and TLS certificates requires careful configuration.

### 11.1 Environment Variables

The simplest method is to use environment variables. In your YAML configuration, you can reference them using the `${VAR_NAME}` syntax.

```yaml
database:
  connection_string: "postgres://${DB_USER}:${DB_PASSWORD}@db.internal:5432/manus"
```

### 11.2 External Secret Stores

For enterprise deployments, integrating with an external secret store like HashiCorp Vault or AWS Secrets Manager is recommended.

- **`secrets.provider`** (String)
  - **Description**: The secret management provider. Supported values: `env`, `vault`, `aws_secrets_manager`.
  - **Default**: `env`

- **`secrets.vault.address`** (String)
  - **Description**: The URL of the Vault server.
  - **Required**: If provider is `vault`.

- **`secrets.vault.token`** (String)
  - **Description**: The Vault authentication token.
  - **Required**: If provider is `vault`.

By centralizing secret management, you reduce the risk of credential leakage and simplify rotation policies.

## 12. Schema Evolution and Versioning

As your workflows and integrations evolve, so too will your configuration schemas. manus-workflows supports schema versioning to ensure backward compatibility.

### 12.1 The `version` Field

Every workflow definition and global configuration file should include a `version` field. This allows the engine to apply the correct parsing logic and default values.

### 12.2 Deprecation Policy

When a configuration field is deprecated, the engine will emit a `WARN` level log message during startup or workflow validation. It is best practice to monitor these logs and update your configurations before the deprecated fields are removed in a subsequent major release.

## 13. Final Thoughts

Configuration management is an ongoing process. As your usage of manus-workflows scales, you will need to continuously monitor performance metrics and adjust your configurations accordingly. By leveraging the comprehensive schemas detailed in this guide, you can build a resilient, secure, and highly performant automation platform.
