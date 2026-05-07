# VoIP On-Call Configuration Schemas Guide

## 1. Introduction

Welcome to the comprehensive Configuration Schemas Guide for the VoIP On-Call system. This document provides an exhaustive, deep-dive reference into every configuration file, schema definition, field, default value, and best practice required to operate, maintain, and scale the VoIP On-Call infrastructure. As enterprise communication systems become increasingly complex, ensuring that your on-call routing, escalation policies, SIP trunking, and notification gateways are correctly configured is paramount to maintaining high availability and meeting stringent Service Level Agreements (SLAs).

This guide is intended for senior system administrators, site reliability engineers (SREs), and VoIP architects who are responsible for deploying and managing the VoIP On-Call system. By the end of this document, you will have a profound understanding of the underlying JSON and YAML schemas that drive the system's behavior, enabling you to fine-tune performance, ensure security, and implement robust disaster recovery strategies.

## 2. System Architecture Overview

Before diving into the specific configuration schemas, it is essential to understand the high-level architecture of the VoIP On-Call system. The system is composed of several microservices, each responsible for a distinct domain of functionality:

- **SIP Gateway Service**: Handles incoming and outgoing SIP signaling and media streams.
- **Routing Engine**: Determines the optimal path for incoming calls based on on-call schedules, escalation policies, and agent availability.
- **Notification Service**: Dispatches SMS, email, and push notifications to on-call personnel.
- **Configuration Management API**: Provides a centralized interface for updating and validating configuration schemas across all services.

Each of these services relies on specific configuration files, which are typically stored in a centralized configuration repository (e.g., Git) and deployed via CI/CD pipelines or configuration management tools like Ansible, Chef, or Puppet.

## 3. Core Configuration Schemas

The VoIP On-Call system utilizes JSON Schema (Draft 7) to validate all configuration files. This ensures that any changes made to the configuration are syntactically correct and semantically valid before they are applied to the production environment.

### 3.1. Global System Configuration (`global.yaml`)

The `global.yaml` file contains settings that apply to the entire VoIP On-Call cluster. This includes logging levels, database connection strings, and global feature flags.

#### Schema Definition

```yaml
$schema: "http://json-schema.org/draft-07/schema#"
title: "Global System Configuration"
type: "object"
properties:
  environment:
    type: "string"
    enum: ["development", "staging", "production"]
    description: "The deployment environment."
  log_level:
    type: "string"
    enum: ["DEBUG", "INFO", "WARN", "ERROR", "FATAL"]
    default: "INFO"
    description: "The global logging level for all services."
  database:
    type: "object"
    properties:
      host:
        type: "string"
        description: "The hostname or IP address of the primary database."
      port:
        type: "integer"
        default: 5432
        description: "The port number of the primary database."
      username:
        type: "string"
        description: "The database user."
      password_secret_arn:
        type: "string"
        description: "The AWS Secrets Manager ARN containing the database password."
    required: ["host", "username", "password_secret_arn"]
required: ["environment", "database"]
```

#### Field Descriptions and Best Practices

- **`environment`**: Always explicitly set this to `production` in live environments to enable strict validation and disable debug endpoints.
- **`log_level`**: In production, `INFO` is recommended to balance visibility with performance. Use `DEBUG` only during active troubleshooting, as it can generate excessive I/O and consume significant disk space.
- **`database.password_secret_arn`**: Never hardcode passwords in the configuration file. Always use a secrets management solution (e.g., AWS Secrets Manager, HashiCorp Vault) and reference the secret via its ARN or path.

### 3.2. SIP Trunk Configuration (`sip_trunks.json`)

The `sip_trunks.json` file defines the connections to external SIP providers (ITSPs) or internal PBX systems. This is a critical configuration, as misconfigurations here can lead to dropped calls or security vulnerabilities.

#### Schema Definition

```json
{
  "$schema": "http://json-schema.org/draft-07/schema#",
  "title": "SIP Trunk Configuration",
  "type": "array",
  "items": {
    "type": "object",
    "properties": {
      "trunk_id": {
        "type": "string",
        "pattern": "^[a-zA-Z0-9_-]+$",
        "description": "A unique identifier for the SIP trunk."
      },
      "provider_name": {
        "type": "string",
        "description": "The name of the ITSP."
      },
      "host": {
        "type": "string",
        "format": "hostname",
        "description": "The FQDN or IP address of the SIP provider's SBC."
      },
      "port": {
        "type": "integer",
        "default": 5060,
        "minimum": 1,
        "maximum": 65535
      },
      "transport": {
        "type": "string",
        "enum": ["udp", "tcp", "tls"],
        "default": "tls"
      },
      "authentication": {
        "type": "object",
        "properties": {
          "username": { "type": "string" },
          "password_secret_arn": { "type": "string" }
        },
        "required": ["username", "password_secret_arn"]
      },
      "codecs": {
        "type": "array",
        "items": {
          "type": "string",
          "enum": ["PCMU", "PCMA", "G722", "G729", "OPUS"]
        },
        "default": ["PCMU", "PCMA"]
      }
    },
    "required": ["trunk_id", "host", "transport"]
  }
}
```

#### Field Descriptions and Best Practices

- **`transport`**: It is highly recommended to use `tls` for all SIP signaling to prevent eavesdropping and man-in-the-middle attacks. If `udp` or `tcp` must be used for legacy compatibility, ensure the traffic is routed over a secure VPN or dedicated direct connect.
- **`codecs`**: Order matters. The system will negotiate codecs in the order they are listed. Place high-definition codecs like `G722` or `OPUS` first if bandwidth permits, falling back to `PCMU`/`PCMA` for broader compatibility.

### 3.3. On-Call Schedule Configuration (`schedules.yaml`)

The `schedules.yaml` file defines the shifts, rotations, and overrides for on-call personnel. This schema is highly complex due to the nature of time zones, recurring events, and exception handling.

#### Schema Definition

```yaml
$schema: "http://json-schema.org/draft-07/schema#"
title: "On-Call Schedules"
type: "array"
items:
  type: "object"
  properties:
    schedule_id:
      type: "string"
      description: "Unique identifier for the schedule."
    name:
      type: "string"
      description: "Human-readable name of the schedule."
    time_zone:
      type: "string"
      description: "IANA Time Zone database name (e.g., America/New_York)."
    layers:
      type: "array"
      description: "Layers of rotations that make up the schedule."
      items:
        type: "object"
        properties:
          layer_id:
            type: "string"
          users:
            type: "array"
            items:
              type: "string"
              description: "User IDs participating in this layer."
          rotation_type:
            type: "string"
            enum: ["daily", "weekly", "custom"]
          shift_length_hours:
            type: "integer"
            minimum: 1
          start_time:
            type: "string"
            format: "date-time"
            description: "The anchor point for the rotation."
        required: ["layer_id", "users", "rotation_type", "start_time"]
  required: ["schedule_id", "name", "time_zone", "layers"]
```

#### Field Descriptions and Best Practices

- **`time_zone`**: Always use IANA time zone names (e.g., `Europe/London`) rather than abbreviations (e.g., `EST`, `BST`), as abbreviations are often ambiguous and do not account for Daylight Saving Time transitions correctly.
- **`layers`**: Use multiple layers to handle complex scenarios, such as a primary rotation that changes weekly and a secondary "shadow" rotation that changes daily. The system evaluates layers from top to bottom, with higher layers taking precedence.

### 3.4. Escalation Policy Configuration (`escalations.json`)

Escalation policies dictate what happens when an incoming call is not answered by the primary on-call person. The `escalations.json` schema defines the rules for routing the call to secondary responders, managers, or voicemail.

#### Schema Definition

```json
{
  "$schema": "http://json-schema.org/draft-07/schema#",
  "title": "Escalation Policies",
  "type": "array",
  "items": {
    "type": "object",
    "properties": {
      "policy_id": {
        "type": "string"
      },
      "name": {
        "type": "string"
      },
      "rules": {
        "type": "array",
        "items": {
          "type": "object",
          "properties": {
            "delay_seconds": {
              "type": "integer",
              "minimum": 0,
              "description": "Time to wait before executing this rule."
            },
            "targets": {
              "type": "array",
              "items": {
                "type": "object",
                "properties": {
                  "target_type": {
                    "type": "string",
                    "enum": ["user", "schedule", "voicemail", "webhook"]
                  },
                  "target_id": {
                    "type": "string"
                  }
                },
                "required": ["target_type", "target_id"]
              }
            }
          },
          "required": ["delay_seconds", "targets"]
        }
      },
      "repeat_count": {
        "type": "integer",
        "default": 0,
        "description": "Number of times to repeat the entire policy if unacknowledged."
      }
    },
    "required": ["policy_id", "name", "rules"]
  }
}
```

#### Field Descriptions and Best Practices

- **`delay_seconds`**: The first rule should typically have a `delay_seconds` of `0` to immediately route the call. Subsequent rules should have delays (e.g., `300` for 5 minutes) to give the previous target time to respond.
- **`repeat_count`**: Be cautious with high repeat counts, as they can lead to alert fatigue. A common best practice is to repeat the policy 1-2 times before finally routing to a fallback mechanism, such as a manager or a third-party answering service.

## 4. Advanced Configuration Topics

### 4.1. Dynamic Configuration Reloading

The VoIP On-Call system supports dynamic reloading of certain configuration files without requiring a service restart. This is achieved via a `SIGHUP` signal or an API endpoint (`POST /api/v1/config/reload`).

Files that support dynamic reloading:
- `schedules.yaml`
- `escalations.json`
- `routing_rules.yaml`

Files that require a service restart:
- `global.yaml` (specifically database and port bindings)
- `sip_trunks.json` (requires re-registration with ITSPs)

### 4.2. Schema Validation in CI/CD

To prevent invalid configurations from causing production outages, it is mandatory to integrate schema validation into your CI/CD pipeline. We provide a CLI tool, `voip-config-validator`, which can be run as a pre-commit hook or a CI step.

Example usage:
```bash
voip-config-validator validate --schema schemas/sip_trunks.schema.json --file config/production/sip_trunks.json
```

### 4.3. Managing Secrets

As mentioned in the `global.yaml` section, secrets must never be stored in plaintext. The system natively integrates with AWS Secrets Manager, HashiCorp Vault, and Kubernetes Secrets.

When defining a secret in the configuration, use the `_secret_arn` or `_secret_path` suffix convention. The configuration loader will automatically resolve these references at startup.

Example:
```yaml
twilio_api_key_secret_path: "secret/data/voip-oncall/twilio/api_key"
```

## 5. Troubleshooting Configuration Issues

When configuration issues arise, the system provides several mechanisms for diagnosis:

1.  **Startup Logs**: If a service fails to start, check the `FATAL` logs. The configuration loader will output detailed JSON Schema validation errors, including the exact file, line number, and violated constraint.
2.  **Configuration Dump Endpoint**: You can retrieve the currently loaded, fully resolved configuration (with secrets redacted) via `GET /api/v1/config/dump`. This is useful for verifying that dynamic reloads were successful.
3.  **Dry Run Mode**: You can start the services with the `--dry-run` flag. This will load and validate all configurations, attempt to connect to external dependencies (like databases), and then exit with a status code of `0` if successful, or `1` if errors were found.

## 6. Conclusion

Mastering the configuration schemas of the VoIP On-Call system is crucial for building a resilient, scalable, and secure communication platform. By adhering to the schemas defined in this guide, utilizing strict validation in your deployment pipelines, and following the outlined best practices, you can ensure that your on-call routing and notifications operate flawlessly, even under the most demanding conditions.

Always refer back to this documentation when introducing new SIP trunks, modifying complex escalation policies, or upgrading the system to a new major version, as schemas may evolve to support new features and capabilities.


## Part 2: Extended Deep Dive

Welcome to the comprehensive Configuration Schemas Guide for the VoIP On-Call system. This document provides an exhaustive, deep-dive reference into every configuration file, schema definition, field, default value, and best practice required to operate, maintain, and scale the VoIP On-Call infrastructure. As enterprise communication systems become increasingly complex, ensuring that your on-call routing, escalation policies, SIP trunking, and notification gateways are correctly configured is paramount to maintaining high availability and meeting stringent Service Level Agreements (SLAs).

This guide is intended for senior system administrators, site reliability engineers (SREs), and VoIP architects who are responsible for deploying and managing the VoIP On-Call system. By the end of this document, you will have a profound understanding of the underlying JSON and YAML schemas that drive the system's behavior, enabling you to fine-tune performance, ensure security, and implement robust disaster recovery strategies.

## 2. System Architecture Overview

Before diving into the specific configuration schemas, it is essential to understand the high-level architecture of the VoIP On-Call system. The system is composed of several microservices, each responsible for a distinct domain of functionality:

- **SIP Gateway Service**: Handles incoming and outgoing SIP signaling and media streams.
- **Routing Engine**: Determines the optimal path for incoming calls based on on-call schedules, escalation policies, and agent availability.
- **Notification Service**: Dispatches SMS, email, and push notifications to on-call personnel.
- **Configuration Management API**: Provides a centralized interface for updating and validating configuration schemas across all services.

Each of these services relies on specific configuration files, which are typically stored in a centralized configuration repository (e.g., Git) and deployed via CI/CD pipelines or configuration management tools like Ansible, Chef, or Puppet.

## 3. Core Configuration Schemas

The VoIP On-Call system utilizes JSON Schema (Draft 7) to validate all configuration files. This ensures that any changes made to the configuration are syntactically correct and semantically valid before they are applied to the production environment.

### 3.1. Global System Configuration (`global.yaml`)

The `global.yaml` file contains settings that apply to the entire VoIP On-Call cluster. This includes logging levels, database connection strings, and global feature flags.

#### Schema Definition

```yaml
$schema: "http://json-schema.org/draft-07/schema#"
title: "Global System Configuration"
type: "object"
properties:
  environment:
    type: "string"
    enum: ["development", "staging", "production"]
    description: "The deployment environment."
  log_level:
    type: "string"
    enum: ["DEBUG", "INFO", "WARN", "ERROR", "FATAL"]
    default: "INFO"
    description: "The global logging level for all services."
  database:
    type: "object"
    properties:
      host:
        type: "string"
        description: "The hostname or IP address of the primary database."
      port:
        type: "integer"
        default: 5432
        description: "The port number of the primary database."
      username:
        type: "string"
        description: "The database user."
      password_secret_arn:
        type: "string"
        description: "The AWS Secrets Manager ARN containing the database password."
    required: ["host", "username", "password_secret_arn"]
required: ["environment", "database"]
```

#### Field Descriptions and Best Practices

- **`environment`**: Always explicitly set this to `production` in live environments to enable strict validation and disable debug endpoints.
- **`log_level`**: In production, `INFO` is recommended to balance visibility with performance. Use `DEBUG` only during active troubleshooting, as it can generate excessive I/O and consume significant disk space.
- **`database.password_secret_arn`**: Never hardcode passwords in the configuration file. Always use a secrets management solution (e.g., AWS Secrets Manager, HashiCorp Vault) and reference the secret via its ARN or path.

### 3.2. SIP Trunk Configuration (`sip_trunks.json`)

The `sip_trunks.json` file defines the connections to external SIP providers (ITSPs) or internal PBX systems. This is a critical configuration, as misconfigurations here can lead to dropped calls or security vulnerabilities.

#### Schema Definition

```json
{
  "$schema": "http://json-schema.org/draft-07/schema#",
  "title": "SIP Trunk Configuration",
  "type": "array",
  "items": {
    "type": "object",
    "properties": {
      "trunk_id": {
        "type": "string",
        "pattern": "^[a-zA-Z0-9_-]+$",
        "description": "A unique identifier for the SIP trunk."
      },
      "provider_name": {
        "type": "string",
        "description": "The name of the ITSP."
      },
      "host": {
        "type": "string",
        "format": "hostname",
        "description": "The FQDN or IP address of the SIP provider's SBC."
      },
      "port": {
        "type": "integer",
        "default": 5060,
        "minimum": 1,
        "maximum": 65535
      },
      "transport": {
        "type": "string",
        "enum": ["udp", "tcp", "tls"],
        "default": "tls"
      },
      "authentication": {
        "type": "object",
        "properties": {
          "username": { "type": "string" },
          "password_secret_arn": { "type": "string" }
        },
        "required": ["username", "password_secret_arn"]
      },
      "codecs": {
        "type": "array",
        "items": {
          "type": "string",
          "enum": ["PCMU", "PCMA", "G722", "G729", "OPUS"]
        },
        "default": ["PCMU", "PCMA"]
      }
    },
    "required": ["trunk_id", "host", "transport"]
  }
}
```

#### Field Descriptions and Best Practices

- **`transport`**: It is highly recommended to use `tls` for all SIP signaling to prevent eavesdropping and man-in-the-middle attacks. If `udp` or `tcp` must be used for legacy compatibility, ensure the traffic is routed over a secure VPN or dedicated direct connect.
- **`codecs`**: Order matters. The system will negotiate codecs in the order they are listed. Place high-definition codecs like `G722` or `OPUS` first if bandwidth permits, falling back to `PCMU`/`PCMA` for broader compatibility.

### 3.3. On-Call Schedule Configuration (`schedules.yaml`)

The `schedules.yaml` file defines the shifts, rotations, and overrides for on-call personnel. This schema is highly complex due to the nature of time zones, recurring events, and exception handling.

#### Schema Definition

```yaml
$schema: "http://json-schema.org/draft-07/schema#"
title: "On-Call Schedules"
type: "array"
items:
  type: "object"
  properties:
    schedule_id:
      type: "string"
      description: "Unique identifier for the schedule."
    name:
      type: "string"
      description: "Human-readable name of the schedule."
    time_zone:
      type: "string"
      description: "IANA Time Zone database name (e.g., America/New_York)."
    layers:
      type: "array"
      description: "Layers of rotations that make up the schedule."
      items:
        type: "object"
        properties:
          layer_id:
            type: "string"
          users:
            type: "array"
            items:
              type: "string"
              description: "User IDs participating in this layer."
          rotation_type:
            type: "string"
            enum: ["daily", "weekly", "custom"]
          shift_length_hours:
            type: "integer"
            minimum: 1
          start_time:
            type: "string"
            format: "date-time"
            description: "The anchor point for the rotation."
        required: ["layer_id", "users", "rotation_type", "start_time"]
  required: ["schedule_id", "name", "time_zone", "layers"]
```

#### Field Descriptions and Best Practices

- **`time_zone`**: Always use IANA time zone names (e.g., `Europe/London`) rather than abbreviations (e.g., `EST`, `BST`), as abbreviations are often ambiguous and do not account for Daylight Saving Time transitions correctly.
- **`layers`**: Use multiple layers to handle complex scenarios, such as a primary rotation that changes weekly and a secondary "shadow" rotation that changes daily. The system evaluates layers from top to bottom, with higher layers taking precedence.

### 3.4. Escalation Policy Configuration (`escalations.json`)

Escalation policies dictate what happens when an incoming call is not answered by the primary on-call person. The `escalations.json` schema defines the rules for routing the call to secondary responders, managers, or voicemail.

#### Schema Definition

```json
{
  "$schema": "http://json-schema.org/draft-07/schema#",
  "title": "Escalation Policies",
  "type": "array",
  "items": {
    "type": "object",
    "properties": {
      "policy_id": {
        "type": "string"
      },
      "name": {
        "type": "string"
      },
      "rules": {
        "type": "array",
        "items": {
          "type": "object",
          "properties": {
            "delay_seconds": {
              "type": "integer",
              "minimum": 0,
              "description": "Time to wait before executing this rule."
            },
            "targets": {
              "type": "array",
              "items": {
                "type": "object",
                "properties": {
                  "target_type": {
                    "type": "string",
                    "enum": ["user", "schedule", "voicemail", "webhook"]
                  },
                  "target_id": {
                    "type": "string"
                  }
                },
                "required": ["target_type", "target_id"]
              }
            }
          },
          "required": ["delay_seconds", "targets"]
        }
      },
      "repeat_count": {
        "type": "integer",
        "default": 0,
        "description": "Number of times to repeat the entire policy if unacknowledged."
      }
    },
    "required": ["policy_id", "name", "rules"]
  }
}
```

#### Field Descriptions and Best Practices

- **`delay_seconds`**: The first rule should typically have a `delay_seconds` of `0` to immediately route the call. Subsequent rules should have delays (e.g., `300` for 5 minutes) to give the previous target time to respond.
- **`repeat_count`**: Be cautious with high repeat counts, as they can lead to alert fatigue. A common best practice is to repeat the policy 1-2 times before finally routing to a fallback mechanism, such as a manager or a third-party answering service.

## 4. Advanced Configuration Topics

### 4.1. Dynamic Configuration Reloading

The VoIP On-Call system supports dynamic reloading of certain configuration files without requiring a service restart. This is achieved via a `SIGHUP` signal or an API endpoint (`POST /api/v1/config/reload`).

Files that support dynamic reloading:
- `schedules.yaml`
- `escalations.json`
- `routing_rules.yaml`

Files that require a service restart:
- `global.yaml` (specifically database and port bindings)
- `sip_trunks.json` (requires re-registration with ITSPs)

### 4.2. Schema Validation in CI/CD

To prevent invalid configurations from causing production outages, it is mandatory to integrate schema validation into your CI/CD pipeline. We provide a CLI tool, `voip-config-validator`, which can be run as a pre-commit hook or a CI step.

Example usage:
```bash
voip-config-validator validate --schema schemas/sip_trunks.schema.json --file config/production/sip_trunks.json
```

### 4.3. Managing Secrets

As mentioned in the `global.yaml` section, secrets must never be stored in plaintext. The system natively integrates with AWS Secrets Manager, HashiCorp Vault, and Kubernetes Secrets.

When defining a secret in the configuration, use the `_secret_arn` or `_secret_path` suffix convention. The configuration loader will automatically resolve these references at startup.

Example:
```yaml
twilio_api_key_secret_path: "secret/data/voip-oncall/twilio/api_key"
```

## 5. Troubleshooting Configuration Issues

When configuration issues arise, the system provides several mechanisms for diagnosis:

1.  **Startup Logs**: If a service fails to start, check the `FATAL` logs. The configuration loader will output detailed JSON Schema validation errors, including the exact file, line number, and violated constraint.
2.  **Configuration Dump Endpoint**: You can retrieve the currently loaded, fully resolved configuration (with secrets redacted) via `GET /api/v1/config/dump`. This is useful for verifying that dynamic reloads were successful.
3.  **Dry Run Mode**: You can start the services with the `--dry-run` flag. This will load and validate all configurations, attempt to connect to external dependencies (like databases), and then exit with a status code of `0` if successful, or `1` if errors were found.

## 6. Conclusion

Mastering the configuration schemas of the VoIP On-Call system is crucial for building a resilient, scalable, and secure communication platform. By adhering to the schemas defined in this guide, utilizing strict validation in your deployment pipelines, and following the outlined best practices, you can ensure that your on-call routing and notifications operate flawlessly, even under the most demanding conditions.

Always refer back to this documentation when introducing new SIP trunks, modifying complex escalation policies, or upgrading the system to a new major version, as schemas may evolve to support new features and capabilities.


## Part 3: Extended Deep Dive

Welcome to the comprehensive Configuration Schemas Guide for the VoIP On-Call system. This document provides an exhaustive, deep-dive reference into every configuration file, schema definition, field, default value, and best practice required to operate, maintain, and scale the VoIP On-Call infrastructure. As enterprise communication systems become increasingly complex, ensuring that your on-call routing, escalation policies, SIP trunking, and notification gateways are correctly configured is paramount to maintaining high availability and meeting stringent Service Level Agreements (SLAs).

This guide is intended for senior system administrators, site reliability engineers (SREs), and VoIP architects who are responsible for deploying and managing the VoIP On-Call system. By the end of this document, you will have a profound understanding of the underlying JSON and YAML schemas that drive the system's behavior, enabling you to fine-tune performance, ensure security, and implement robust disaster recovery strategies.

## 2. System Architecture Overview

Before diving into the specific configuration schemas, it is essential to understand the high-level architecture of the VoIP On-Call system. The system is composed of several microservices, each responsible for a distinct domain of functionality:

- **SIP Gateway Service**: Handles incoming and outgoing SIP signaling and media streams.
- **Routing Engine**: Determines the optimal path for incoming calls based on on-call schedules, escalation policies, and agent availability.
- **Notification Service**: Dispatches SMS, email, and push notifications to on-call personnel.
- **Configuration Management API**: Provides a centralized interface for updating and validating configuration schemas across all services.

Each of these services relies on specific configuration files, which are typically stored in a centralized configuration repository (e.g., Git) and deployed via CI/CD pipelines or configuration management tools like Ansible, Chef, or Puppet.

## 3. Core Configuration Schemas

The VoIP On-Call system utilizes JSON Schema (Draft 7) to validate all configuration files. This ensures that any changes made to the configuration are syntactically correct and semantically valid before they are applied to the production environment.

### 3.1. Global System Configuration (`global.yaml`)

The `global.yaml` file contains settings that apply to the entire VoIP On-Call cluster. This includes logging levels, database connection strings, and global feature flags.

#### Schema Definition

```yaml
$schema: "http://json-schema.org/draft-07/schema#"
title: "Global System Configuration"
type: "object"
properties:
  environment:
    type: "string"
    enum: ["development", "staging", "production"]
    description: "The deployment environment."
  log_level:
    type: "string"
    enum: ["DEBUG", "INFO", "WARN", "ERROR", "FATAL"]
    default: "INFO"
    description: "The global logging level for all services."
  database:
    type: "object"
    properties:
      host:
        type: "string"
        description: "The hostname or IP address of the primary database."
      port:
        type: "integer"
        default: 5432
        description: "The port number of the primary database."
      username:
        type: "string"
        description: "The database user."
      password_secret_arn:
        type: "string"
        description: "The AWS Secrets Manager ARN containing the database password."
    required: ["host", "username", "password_secret_arn"]
required: ["environment", "database"]
```

#### Field Descriptions and Best Practices

- **`environment`**: Always explicitly set this to `production` in live environments to enable strict validation and disable debug endpoints.
- **`log_level`**: In production, `INFO` is recommended to balance visibility with performance. Use `DEBUG` only during active troubleshooting, as it can generate excessive I/O and consume significant disk space.
- **`database.password_secret_arn`**: Never hardcode passwords in the configuration file. Always use a secrets management solution (e.g., AWS Secrets Manager, HashiCorp Vault) and reference the secret via its ARN or path.

### 3.2. SIP Trunk Configuration (`sip_trunks.json`)

The `sip_trunks.json` file defines the connections to external SIP providers (ITSPs) or internal PBX systems. This is a critical configuration, as misconfigurations here can lead to dropped calls or security vulnerabilities.

#### Schema Definition

```json
{
  "$schema": "http://json-schema.org/draft-07/schema#",
  "title": "SIP Trunk Configuration",
  "type": "array",
  "items": {
    "type": "object",
    "properties": {
      "trunk_id": {
        "type": "string",
        "pattern": "^[a-zA-Z0-9_-]+$",
        "description": "A unique identifier for the SIP trunk."
      },
      "provider_name": {
        "type": "string",
        "description": "The name of the ITSP."
      },
      "host": {
        "type": "string",
        "format": "hostname",
        "description": "The FQDN or IP address of the SIP provider's SBC."
      },
      "port": {
        "type": "integer",
        "default": 5060,
        "minimum": 1,
        "maximum": 65535
      },
      "transport": {
        "type": "string",
        "enum": ["udp", "tcp", "tls"],
        "default": "tls"
      },
      "authentication": {
        "type": "object",
        "properties": {
          "username": { "type": "string" },
          "password_secret_arn": { "type": "string" }
        },
        "required": ["username", "password_secret_arn"]
      },
      "codecs": {
        "type": "array",
        "items": {
          "type": "string",
          "enum": ["PCMU", "PCMA", "G722", "G729", "OPUS"]
        },
        "default": ["PCMU", "PCMA"]
      }
    },
    "required": ["trunk_id", "host", "transport"]
  }
}
```

#### Field Descriptions and Best Practices

- **`transport`**: It is highly recommended to use `tls` for all SIP signaling to prevent eavesdropping and man-in-the-middle attacks. If `udp` or `tcp` must be used for legacy compatibility, ensure the traffic is routed over a secure VPN or dedicated direct connect.
- **`codecs`**: Order matters. The system will negotiate codecs in the order they are listed. Place high-definition codecs like `G722` or `OPUS` first if bandwidth permits, falling back to `PCMU`/`PCMA` for broader compatibility.

### 3.3. On-Call Schedule Configuration (`schedules.yaml`)

The `schedules.yaml` file defines the shifts, rotations, and overrides for on-call personnel. This schema is highly complex due to the nature of time zones, recurring events, and exception handling.

#### Schema Definition

```yaml
$schema: "http://json-schema.org/draft-07/schema#"
title: "On-Call Schedules"
type: "array"
items:
  type: "object"
  properties:
    schedule_id:
      type: "string"
      description: "Unique identifier for the schedule."
    name:
      type: "string"
      description: "Human-readable name of the schedule."
    time_zone:
      type: "string"
      description: "IANA Time Zone database name (e.g., America/New_York)."
    layers:
      type: "array"
      description: "Layers of rotations that make up the schedule."
      items:
        type: "object"
        properties:
          layer_id:
            type: "string"
          users:
            type: "array"
            items:
              type: "string"
              description: "User IDs participating in this layer."
          rotation_type:
            type: "string"
            enum: ["daily", "weekly", "custom"]
          shift_length_hours:
            type: "integer"
            minimum: 1
          start_time:
            type: "string"
            format: "date-time"
            description: "The anchor point for the rotation."
        required: ["layer_id", "users", "rotation_type", "start_time"]
  required: ["schedule_id", "name", "time_zone", "layers"]
```

#### Field Descriptions and Best Practices

- **`time_zone`**: Always use IANA time zone names (e.g., `Europe/London`) rather than abbreviations (e.g., `EST`, `BST`), as abbreviations are often ambiguous and do not account for Daylight Saving Time transitions correctly.
- **`layers`**: Use multiple layers to handle complex scenarios, such as a primary rotation that changes weekly and a secondary "shadow" rotation that changes daily. The system evaluates layers from top to bottom, with higher layers taking precedence.

### 3.4. Escalation Policy Configuration (`escalations.json`)

Escalation policies dictate what happens when an incoming call is not answered by the primary on-call person. The `escalations.json` schema defines the rules for routing the call to secondary responders, managers, or voicemail.

#### Schema Definition

```json
{
  "$schema": "http://json-schema.org/draft-07/schema#",
  "title": "Escalation Policies",
  "type": "array",
  "items": {
    "type": "object",
    "properties": {
      "policy_id": {
        "type": "string"
      },
      "name": {
        "type": "string"
      },
      "rules": {
        "type": "array",
        "items": {
          "type": "object",
          "properties": {
            "delay_seconds": {
              "type": "integer",
              "minimum": 0,
              "description": "Time to wait before executing this rule."
            },
            "targets": {
              "type": "array",
              "items": {
                "type": "object",
                "properties": {
                  "target_type": {
                    "type": "string",
                    "enum": ["user", "schedule", "voicemail", "webhook"]
                  },
                  "target_id": {
                    "type": "string"
                  }
                },
                "required": ["target_type", "target_id"]
              }
            }
          },
          "required": ["delay_seconds", "targets"]
        }
      },
      "repeat_count": {
        "type": "integer",
        "default": 0,
        "description": "Number of times to repeat the entire policy if unacknowledged."
      }
    },
    "required": ["policy_id", "name", "rules"]
  }
}
```

#### Field Descriptions and Best Practices

- **`delay_seconds`**: The first rule should typically have a `delay_seconds` of `0` to immediately route the call. Subsequent rules should have delays (e.g., `300` for 5 minutes) to give the previous target time to respond.
- **`repeat_count`**: Be cautious with high repeat counts, as they can lead to alert fatigue. A common best practice is to repeat the policy 1-2 times before finally routing to a fallback mechanism, such as a manager or a third-party answering service.

## 4. Advanced Configuration Topics

### 4.1. Dynamic Configuration Reloading

The VoIP On-Call system supports dynamic reloading of certain configuration files without requiring a service restart. This is achieved via a `SIGHUP` signal or an API endpoint (`POST /api/v1/config/reload`).

Files that support dynamic reloading:
- `schedules.yaml`
- `escalations.json`
- `routing_rules.yaml`

Files that require a service restart:
- `global.yaml` (specifically database and port bindings)
- `sip_trunks.json` (requires re-registration with ITSPs)

### 4.2. Schema Validation in CI/CD

To prevent invalid configurations from causing production outages, it is mandatory to integrate schema validation into your CI/CD pipeline. We provide a CLI tool, `voip-config-validator`, which can be run as a pre-commit hook or a CI step.

Example usage:
```bash
voip-config-validator validate --schema schemas/sip_trunks.schema.json --file config/production/sip_trunks.json
```

### 4.3. Managing Secrets

As mentioned in the `global.yaml` section, secrets must never be stored in plaintext. The system natively integrates with AWS Secrets Manager, HashiCorp Vault, and Kubernetes Secrets.

When defining a secret in the configuration, use the `_secret_arn` or `_secret_path` suffix convention. The configuration loader will automatically resolve these references at startup.

Example:
```yaml
twilio_api_key_secret_path: "secret/data/voip-oncall/twilio/api_key"
```

## 5. Troubleshooting Configuration Issues

When configuration issues arise, the system provides several mechanisms for diagnosis:

1.  **Startup Logs**: If a service fails to start, check the `FATAL` logs. The configuration loader will output detailed JSON Schema validation errors, including the exact file, line number, and violated constraint.
2.  **Configuration Dump Endpoint**: You can retrieve the currently loaded, fully resolved configuration (with secrets redacted) via `GET /api/v1/config/dump`. This is useful for verifying that dynamic reloads were successful.
3.  **Dry Run Mode**: You can start the services with the `--dry-run` flag. This will load and validate all configurations, attempt to connect to external dependencies (like databases), and then exit with a status code of `0` if successful, or `1` if errors were found.

## 6. Conclusion

Mastering the configuration schemas of the VoIP On-Call system is crucial for building a resilient, scalable, and secure communication platform. By adhering to the schemas defined in this guide, utilizing strict validation in your deployment pipelines, and following the outlined best practices, you can ensure that your on-call routing and notifications operate flawlessly, even under the most demanding conditions.

Always refer back to this documentation when introducing new SIP trunks, modifying complex escalation policies, or upgrading the system to a new major version, as schemas may evolve to support new features and capabilities.


## Part 4: Extended Deep Dive

Welcome to the comprehensive Configuration Schemas Guide for the VoIP On-Call system. This document provides an exhaustive, deep-dive reference into every configuration file, schema definition, field, default value, and best practice required to operate, maintain, and scale the VoIP On-Call infrastructure. As enterprise communication systems become increasingly complex, ensuring that your on-call routing, escalation policies, SIP trunking, and notification gateways are correctly configured is paramount to maintaining high availability and meeting stringent Service Level Agreements (SLAs).

This guide is intended for senior system administrators, site reliability engineers (SREs), and VoIP architects who are responsible for deploying and managing the VoIP On-Call system. By the end of this document, you will have a profound understanding of the underlying JSON and YAML schemas that drive the system's behavior, enabling you to fine-tune performance, ensure security, and implement robust disaster recovery strategies.

## 2. System Architecture Overview

Before diving into the specific configuration schemas, it is essential to understand the high-level architecture of the VoIP On-Call system. The system is composed of several microservices, each responsible for a distinct domain of functionality:

- **SIP Gateway Service**: Handles incoming and outgoing SIP signaling and media streams.
- **Routing Engine**: Determines the optimal path for incoming calls based on on-call schedules, escalation policies, and agent availability.
- **Notification Service**: Dispatches SMS, email, and push notifications to on-call personnel.
- **Configuration Management API**: Provides a centralized interface for updating and validating configuration schemas across all services.

Each of these services relies on specific configuration files, which are typically stored in a centralized configuration repository (e.g., Git) and deployed via CI/CD pipelines or configuration management tools like Ansible, Chef, or Puppet.

## 3. Core Configuration Schemas

The VoIP On-Call system utilizes JSON Schema (Draft 7) to validate all configuration files. This ensures that any changes made to the configuration are syntactically correct and semantically valid before they are applied to the production environment.

### 3.1. Global System Configuration (`global.yaml`)

The `global.yaml` file contains settings that apply to the entire VoIP On-Call cluster. This includes logging levels, database connection strings, and global feature flags.

#### Schema Definition

```yaml
$schema: "http://json-schema.org/draft-07/schema#"
title: "Global System Configuration"
type: "object"
properties:
  environment:
    type: "string"
    enum: ["development", "staging", "production"]
    description: "The deployment environment."
  log_level:
    type: "string"
    enum: ["DEBUG", "INFO", "WARN", "ERROR", "FATAL"]
    default: "INFO"
    description: "The global logging level for all services."
  database:
    type: "object"
    properties:
      host:
        type: "string"
        description: "The hostname or IP address of the primary database."
      port:
        type: "integer"
        default: 5432
        description: "The port number of the primary database."
      username:
        type: "string"
        description: "The database user."
      password_secret_arn:
        type: "string"
        description: "The AWS Secrets Manager ARN containing the database password."
    required: ["host", "username", "password_secret_arn"]
required: ["environment", "database"]
```

#### Field Descriptions and Best Practices

- **`environment`**: Always explicitly set this to `production` in live environments to enable strict validation and disable debug endpoints.
- **`log_level`**: In production, `INFO` is recommended to balance visibility with performance. Use `DEBUG` only during active troubleshooting, as it can generate excessive I/O and consume significant disk space.
- **`database.password_secret_arn`**: Never hardcode passwords in the configuration file. Always use a secrets management solution (e.g., AWS Secrets Manager, HashiCorp Vault) and reference the secret via its ARN or path.

### 3.2. SIP Trunk Configuration (`sip_trunks.json`)

The `sip_trunks.json` file defines the connections to external SIP providers (ITSPs) or internal PBX systems. This is a critical configuration, as misconfigurations here can lead to dropped calls or security vulnerabilities.

#### Schema Definition

```json
{
  "$schema": "http://json-schema.org/draft-07/schema#",
  "title": "SIP Trunk Configuration",
  "type": "array",
  "items": {
    "type": "object",
    "properties": {
      "trunk_id": {
        "type": "string",
        "pattern": "^[a-zA-Z0-9_-]+$",
        "description": "A unique identifier for the SIP trunk."
      },
      "provider_name": {
        "type": "string",
        "description": "The name of the ITSP."
      },
      "host": {
        "type": "string",
        "format": "hostname",
        "description": "The FQDN or IP address of the SIP provider's SBC."
      },
      "port": {
        "type": "integer",
        "default": 5060,
        "minimum": 1,
        "maximum": 65535
      },
      "transport": {
        "type": "string",
        "enum": ["udp", "tcp", "tls"],
        "default": "tls"
      },
      "authentication": {
        "type": "object",
        "properties": {
          "username": { "type": "string" },
          "password_secret_arn": { "type": "string" }
        },
        "required": ["username", "password_secret_arn"]
      },
      "codecs": {
        "type": "array",
        "items": {
          "type": "string",
          "enum": ["PCMU", "PCMA", "G722", "G729", "OPUS"]
        },
        "default": ["PCMU", "PCMA"]
      }
    },
    "required": ["trunk_id", "host", "transport"]
  }
}
```

#### Field Descriptions and Best Practices

- **`transport`**: It is highly recommended to use `tls` for all SIP signaling to prevent eavesdropping and man-in-the-middle attacks. If `udp` or `tcp` must be used for legacy compatibility, ensure the traffic is routed over a secure VPN or dedicated direct connect.
- **`codecs`**: Order matters. The system will negotiate codecs in the order they are listed. Place high-definition codecs like `G722` or `OPUS` first if bandwidth permits, falling back to `PCMU`/`PCMA` for broader compatibility.

### 3.3. On-Call Schedule Configuration (`schedules.yaml`)

The `schedules.yaml` file defines the shifts, rotations, and overrides for on-call personnel. This schema is highly complex due to the nature of time zones, recurring events, and exception handling.

#### Schema Definition

```yaml
$schema: "http://json-schema.org/draft-07/schema#"
title: "On-Call Schedules"
type: "array"
items:
  type: "object"
  properties:
    schedule_id:
      type: "string"
      description: "Unique identifier for the schedule."
    name:
      type: "string"
      description: "Human-readable name of the schedule."
    time_zone:
      type: "string"
      description: "IANA Time Zone database name (e.g., America/New_York)."
    layers:
      type: "array"
      description: "Layers of rotations that make up the schedule."
      items:
        type: "object"
        properties:
          layer_id:
            type: "string"
          users:
            type: "array"
            items:
              type: "string"
              description: "User IDs participating in this layer."
          rotation_type:
            type: "string"
            enum: ["daily", "weekly", "custom"]
          shift_length_hours:
            type: "integer"
            minimum: 1
          start_time:
            type: "string"
            format: "date-time"
            description: "The anchor point for the rotation."
        required: ["layer_id", "users", "rotation_type", "start_time"]
  required: ["schedule_id", "name", "time_zone", "layers"]
```

#### Field Descriptions and Best Practices

- **`time_zone`**: Always use IANA time zone names (e.g., `Europe/London`) rather than abbreviations (e.g., `EST`, `BST`), as abbreviations are often ambiguous and do not account for Daylight Saving Time transitions correctly.
- **`layers`**: Use multiple layers to handle complex scenarios, such as a primary rotation that changes weekly and a secondary "shadow" rotation that changes daily. The system evaluates layers from top to bottom, with higher layers taking precedence.

### 3.4. Escalation Policy Configuration (`escalations.json`)

Escalation policies dictate what happens when an incoming call is not answered by the primary on-call person. The `escalations.json` schema defines the rules for routing the call to secondary responders, managers, or voicemail.

#### Schema Definition

```json
{
  "$schema": "http://json-schema.org/draft-07/schema#",
  "title": "Escalation Policies",
  "type": "array",
  "items": {
    "type": "object",
    "properties": {
      "policy_id": {
        "type": "string"
      },
      "name": {
        "type": "string"
      },
      "rules": {
        "type": "array",
        "items": {
          "type": "object",
          "properties": {
            "delay_seconds": {
              "type": "integer",
              "minimum": 0,
              "description": "Time to wait before executing this rule."
            },
            "targets": {
              "type": "array",
              "items": {
                "type": "object",
                "properties": {
                  "target_type": {
                    "type": "string",
                    "enum": ["user", "schedule", "voicemail", "webhook"]
                  },
                  "target_id": {
                    "type": "string"
                  }
                },
                "required": ["target_type", "target_id"]
              }
            }
          },
          "required": ["delay_seconds", "targets"]
        }
      },
      "repeat_count": {
        "type": "integer",
        "default": 0,
        "description": "Number of times to repeat the entire policy if unacknowledged."
      }
    },
    "required": ["policy_id", "name", "rules"]
  }
}
```

#### Field Descriptions and Best Practices

- **`delay_seconds`**: The first rule should typically have a `delay_seconds` of `0` to immediately route the call. Subsequent rules should have delays (e.g., `300` for 5 minutes) to give the previous target time to respond.
- **`repeat_count`**: Be cautious with high repeat counts, as they can lead to alert fatigue. A common best practice is to repeat the policy 1-2 times before finally routing to a fallback mechanism, such as a manager or a third-party answering service.

## 4. Advanced Configuration Topics

### 4.1. Dynamic Configuration Reloading

The VoIP On-Call system supports dynamic reloading of certain configuration files without requiring a service restart. This is achieved via a `SIGHUP` signal or an API endpoint (`POST /api/v1/config/reload`).

Files that support dynamic reloading:
- `schedules.yaml`
- `escalations.json`
- `routing_rules.yaml`

Files that require a service restart:
- `global.yaml` (specifically database and port bindings)
- `sip_trunks.json` (requires re-registration with ITSPs)

### 4.2. Schema Validation in CI/CD

To prevent invalid configurations from causing production outages, it is mandatory to integrate schema validation into your CI/CD pipeline. We provide a CLI tool, `voip-config-validator`, which can be run as a pre-commit hook or a CI step.

Example usage:
```bash
voip-config-validator validate --schema schemas/sip_trunks.schema.json --file config/production/sip_trunks.json
```

### 4.3. Managing Secrets

As mentioned in the `global.yaml` section, secrets must never be stored in plaintext. The system natively integrates with AWS Secrets Manager, HashiCorp Vault, and Kubernetes Secrets.

When defining a secret in the configuration, use the `_secret_arn` or `_secret_path` suffix convention. The configuration loader will automatically resolve these references at startup.

Example:
```yaml
twilio_api_key_secret_path: "secret/data/voip-oncall/twilio/api_key"
```

## 5. Troubleshooting Configuration Issues

When configuration issues arise, the system provides several mechanisms for diagnosis:

1.  **Startup Logs**: If a service fails to start, check the `FATAL` logs. The configuration loader will output detailed JSON Schema validation errors, including the exact file, line number, and violated constraint.
2.  **Configuration Dump Endpoint**: You can retrieve the currently loaded, fully resolved configuration (with secrets redacted) via `GET /api/v1/config/dump`. This is useful for verifying that dynamic reloads were successful.
3.  **Dry Run Mode**: You can start the services with the `--dry-run` flag. This will load and validate all configurations, attempt to connect to external dependencies (like databases), and then exit with a status code of `0` if successful, or `1` if errors were found.

## 6. Conclusion

Mastering the configuration schemas of the VoIP On-Call system is crucial for building a resilient, scalable, and secure communication platform. By adhering to the schemas defined in this guide, utilizing strict validation in your deployment pipelines, and following the outlined best practices, you can ensure that your on-call routing and notifications operate flawlessly, even under the most demanding conditions.

Always refer back to this documentation when introducing new SIP trunks, modifying complex escalation policies, or upgrading the system to a new major version, as schemas may evolve to support new features and capabilities.
