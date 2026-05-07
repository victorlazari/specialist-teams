# DevOps Configuration Schemas Guide

## 1. Introduction
In the modern DevOps ecosystem, configuration schemas serve as the foundational blueprints for infrastructure, deployment pipelines, container orchestration, and monitoring systems. A well-defined configuration schema ensures consistency, reproducibility, and security across all environments. This comprehensive guide delves deep into the configuration schemas of the most prominent DevOps tools, documenting every critical configuration file, field, default value, and best practice.

## 2. Infrastructure as Code (IaC) Configuration Schemas

### 2.1 Terraform
Terraform uses HashiCorp Configuration Language (HCL) to define infrastructure. The primary configuration files include `main.tf`, `variables.tf`, and `outputs.tf`.

#### 2.1.1 `main.tf`
The `main.tf` file is the primary entry point for Terraform configurations. It defines the providers, resources, and data sources.

**Schema Fields:**
- `terraform`: The top-level block for configuring Terraform behavior.
  - `required_version` (String): Specifies the required Terraform version. Default: None.
  - `required_providers` (Map): Specifies the required providers and their versions.
  - `backend` (Block): Configures the remote state backend (e.g., `s3`, `gcs`).
- `provider`: Configures a specific provider (e.g., `aws`, `azurerm`).
  - `region` (String): The region to deploy resources. Default: Provider-specific.
  - `alias` (String): An alias for the provider to allow multiple configurations.
- `resource`: Defines an infrastructure object.
  - `type` (String): The type of resource (e.g., `aws_instance`).
  - `name` (String): The local name of the resource.
  - Resource-specific arguments (e.g., `ami`, `instance_type`).

**Example:**
```hcl
terraform {
  required_version = ">= 1.0.0"
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 4.0"
    }
  }
  backend "s3" {
    bucket = "my-terraform-state"
    key    = "prod/terraform.tfstate"
    region = "us-east-1"
  }
}

provider "aws" {
  region = "us-east-1"
}

resource "aws_instance" "web" {
  ami           = "ami-0c55b159cbfafe1f0"
  instance_type = "t2.micro"
  tags = {
    Name = "WebServer"
  }
}
```

#### 2.1.2 `variables.tf`
The `variables.tf` file defines input variables for the Terraform module.

**Schema Fields:**
- `variable`: Defines a new variable.
  - `type` (Type): The type of the variable (e.g., `string`, `number`, `list`, `map`). Default: `any`.
  - `default` (Any): The default value if not provided. Default: None.
  - `description` (String): A description of the variable. Default: None.
  - `sensitive` (Boolean): Whether the variable contains sensitive data. Default: `false`.
  - `validation` (Block): Custom validation rules for the variable.

#### 2.1.3 `outputs.tf`
The `outputs.tf` file defines output values that can be queried or used by other modules.

**Schema Fields:**
- `output`: Defines a new output.
  - `value` (Any): The value to output. Required.
  - `description` (String): A description of the output. Default: None.
  - `sensitive` (Boolean): Whether the output contains sensitive data. Default: `false`.

### 2.2 Pulumi
Pulumi allows defining infrastructure using general-purpose programming languages. The configuration is managed via `Pulumi.yaml` and environment-specific `Pulumi.<stack>.yaml` files.

#### 2.2.1 `Pulumi.yaml`
**Schema Fields:**
- `name` (String): The name of the project. Required.
- `runtime` (String/Object): The runtime language (e.g., `nodejs`, `python`, `go`). Required.
- `description` (String): A description of the project. Default: None.
- `template` (Object): Template metadata.

#### 2.2.2 `Pulumi.<stack>.yaml`
**Schema Fields:**
- `config` (Map): Stack-specific configuration values.
  - `<namespace>:<key>` (String/Object): The configuration key and value. Secure values are encrypted.

## 3. CI/CD Pipeline Configuration Schemas

### 3.1 GitHub Actions
GitHub Actions workflows are defined in YAML files located in the `.github/workflows/` directory.

#### 3.1.1 Workflow YAML Schema
**Schema Fields:**
- `name` (String): The name of the workflow. Default: The file path.
- `on` (String/List/Map): The events that trigger the workflow. Required.
  - `push` (Map): Triggers on push events.
    - `branches` (List): Branches to trigger on.
    - `tags` (List): Tags to trigger on.
    - `paths` (List): File paths to trigger on.
  - `pull_request` (Map): Triggers on pull request events.
  - `schedule` (List): Triggers on a cron schedule.
  - `workflow_dispatch` (Map): Allows manual triggering.
- `env` (Map): Environment variables available to all jobs.
- `jobs` (Map): A map of jobs to run. Required.
  - `<job_id>` (Map): The identifier for the job.
    - `name` (String): The display name of the job.
    - `needs` (String/List): Jobs that must complete before this job runs.
    - `runs-on` (String/List): The runner environment (e.g., `ubuntu-latest`). Required.
    - `environment` (String/Map): The deployment environment.
    - `outputs` (Map): Outputs generated by the job.
    - `env` (Map): Job-specific environment variables.
    - `steps` (List): A list of steps to execute. Required.
      - `name` (String): The name of the step.
      - `uses` (String): An action to run (e.g., `actions/checkout@v3`).
      - `run` (String): A shell command to run.
      - `with` (Map): Input parameters for the action.
      - `env` (Map): Step-specific environment variables.

**Example:**
```yaml
name: CI Pipeline
on:
  push:
    branches: [ "main" ]
  pull_request:
    branches: [ "main" ]

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout code
        uses: actions/checkout@v3
      - name: Set up Node.js
        uses: actions/setup-node@v3
        with:
          node-version: '16'
      - name: Install dependencies
        run: npm ci
      - name: Run tests
        run: npm test
```

### 3.2 GitLab CI
GitLab CI pipelines are defined in the `.gitlab-ci.yml` file at the root of the repository.

#### 3.2.1 `.gitlab-ci.yml` Schema
**Schema Fields:**
- `stages` (List): Defines the order of execution for jobs. Default: `[.pre, build, test, deploy, .post]`.
- `variables` (Map): Global environment variables.
- `default` (Map): Default configuration for all jobs (e.g., `image`, `before_script`).
- `include` (List/Map): Includes external YAML files.
- `<job_name>` (Map): Defines a job.
  - `stage` (String): The stage the job belongs to. Default: `test`.
  - `image` (String/Map): The Docker image to use.
  - `script` (List): Shell commands to execute. Required.
  - `before_script` (List): Commands to run before the `script`.
  - `after_script` (List): Commands to run after the `script`.
  - `rules` (List): Conditions for when the job should run.
  - `artifacts` (Map): Files to attach to the job upon success.
    - `paths` (List): File paths to include.
    - `expire_in` (String): How long to keep the artifacts.
  - `cache` (Map): Files to cache between runs.
  - `dependencies` (List): Jobs to fetch artifacts from.

## 4. Container Orchestration Configuration Schemas

### 4.1 Kubernetes Manifests
Kubernetes resources are defined using YAML manifests. Every manifest shares a common base schema.

#### 4.1.1 Base Schema
**Schema Fields:**
- `apiVersion` (String): The version of the Kubernetes API (e.g., `v1`, `apps/v1`). Required.
- `kind` (String): The type of resource (e.g., `Pod`, `Deployment`, `Service`). Required.
- `metadata` (Map): Data that uniquely identifies the resource. Required.
  - `name` (String): The name of the resource. Required.
  - `namespace` (String): The namespace of the resource. Default: `default`.
  - `labels` (Map): Key-value pairs for organizing resources.
  - `annotations` (Map): Key-value pairs for non-identifying metadata.

#### 4.1.2 Deployment Schema (`apps/v1`)
**Schema Fields:**
- `spec` (Map): The desired state of the Deployment.
  - `replicas` (Integer): The number of desired pods. Default: `1`.
  - `selector` (Map): Label selector for pods. Required.
    - `matchLabels` (Map): Key-value pairs to match pod labels.
  - `template` (Map): The pod template. Required.
    - `metadata` (Map): Pod metadata (must match `selector`).
    - `spec` (Map): Pod specification.
      - `containers` (List): A list of containers to run. Required.
        - `name` (String): The name of the container. Required.
        - `image` (String): The Docker image to run. Required.
        - `ports` (List): Ports to expose.
          - `containerPort` (Integer): The port number.
        - `env` (List): Environment variables.
        - `resources` (Map): Compute resources required.
          - `requests` (Map): Minimum resources (e.g., `cpu: 100m`, `memory: 128Mi`).
          - `limits` (Map): Maximum resources.
        - `livenessProbe` (Map): Health check for container liveness.
        - `readinessProbe` (Map): Health check for container readiness.

#### 4.1.3 Service Schema (`v1`)
**Schema Fields:**
- `spec` (Map): The desired state of the Service.
  - `type` (String): The type of service (`ClusterIP`, `NodePort`, `LoadBalancer`, `ExternalName`). Default: `ClusterIP`.
  - `selector` (Map): Route traffic to pods with these labels.
  - `ports` (List): The ports exposed by the service. Required.
    - `port` (Integer): The port exposed by the service. Required.
    - `targetPort` (Integer/String): The port to access on the pods. Default: `port`.
    - `nodePort` (Integer): The port on each node (for `NodePort` type).

### 4.2 Docker Compose
Docker Compose uses `docker-compose.yml` to define multi-container applications.

#### 4.2.1 `docker-compose.yml` Schema
**Schema Fields:**
- `version` (String): The Compose file format version (e.g., `'3.8'`).
- `services` (Map): Defines the containers to run. Required.
  - `<service_name>` (Map): Configuration for a specific service.
    - `image` (String): The Docker image to use.
    - `build` (String/Map): Configuration for building the image from a Dockerfile.
    - `ports` (List): Port mappings (`HOST:CONTAINER`).
    - `environment` (List/Map): Environment variables.
    - `volumes` (List): Volume mappings (`HOST:CONTAINER`).
    - `depends_on` (List/Map): Dependencies between services.
    - `networks` (List): Networks to join.
    - `restart` (String): Restart policy (`no`, `always`, `on-failure`, `unless-stopped`). Default: `no`.
- `volumes` (Map): Defines named volumes.
- `networks` (Map): Defines custom networks.

## 5. Configuration Management Schemas

### 5.1 Ansible
Ansible uses YAML for playbooks and INI/YAML for inventories.

#### 5.1.1 `ansible.cfg`
The `ansible.cfg` file configures Ansible's behavior. It uses an INI format.

**Schema Sections:**
- `[defaults]`: General defaults.
  - `inventory` (String): Path to the inventory file. Default: `/etc/ansible/hosts`.
  - `remote_user` (String): Default user to connect as.
  - `host_key_checking` (Boolean): Whether to check SSH host keys. Default: `True`.
  - `roles_path` (String): Path to search for roles.
- `[privilege_escalation]`: Settings for `sudo`/`su`.
  - `become` (Boolean): Enable privilege escalation. Default: `False`.
  - `become_method` (String): Method to use (e.g., `sudo`). Default: `sudo`.
  - `become_user` (String): User to become. Default: `root`.

#### 5.1.2 Playbook Schema (`playbook.yml`)
**Schema Fields:**
- A playbook is a list of plays (dictionaries).
  - `name` (String): The name of the play.
  - `hosts` (String/List): The hosts to target. Required.
  - `become` (Boolean): Enable privilege escalation for the play.
  - `vars` (Map): Variables for the play.
  - `tasks` (List): A list of tasks to execute.
    - `name` (String): The name of the task.
    - `<module_name>` (Map): The Ansible module to run (e.g., `apt`, `yum`, `copy`, `template`).
    - `when` (String/List): Conditional execution.
    - `loop` (List): Iterate over a list.
    - `register` (String): Save the task output to a variable.
  - `roles` (List): A list of roles to apply.

## 6. Monitoring and Observability Configuration Schemas

### 6.1 Prometheus
Prometheus is configured via `prometheus.yml`.

#### 6.1.1 `prometheus.yml` Schema
**Schema Fields:**
- `global` (Map): Global configuration.
  - `scrape_interval` (Duration): How frequently to scrape targets. Default: `1m`.
  - `evaluation_interval` (Duration): How frequently to evaluate rules. Default: `1m`.
  - `external_labels` (Map): Labels to add to all time series.
- `rule_files` (List): Paths to rule files (recording and alerting rules).
- `alerting` (Map): Configuration for Alertmanager.
  - `alertmanagers` (List): Alertmanager targets.
- `scrape_configs` (List): Configuration for scraping metrics.
  - `job_name` (String): The name of the job. Required.
  - `scrape_interval` (Duration): Overrides the global scrape interval.
  - `metrics_path` (String): The HTTP resource path to scrape. Default: `/metrics`.
  - `static_configs` (List): Statically configured targets.
    - `targets` (List): A list of endpoints (e.g., `['localhost:9090']`).
    - `labels` (Map): Labels assigned to all metrics scraped from these targets.
  - `kubernetes_sd_configs` (List): Kubernetes service discovery configurations.

### 6.2 Grafana
Grafana configuration is typically managed via `grafana.ini` or environment variables, and provisioning files for dashboards and datasources.

#### 6.2.1 Datasource Provisioning (`datasources.yml`)
**Schema Fields:**
- `apiVersion` (Integer): The API version. Default: `1`.
- `datasources` (List): A list of datasources.
  - `name` (String): The name of the datasource. Required.
  - `type` (String): The type of datasource (e.g., `prometheus`, `loki`). Required.
  - `access` (String): Access mode (`proxy` or `direct`). Default: `proxy`.
  - `url` (String): The URL of the datasource. Required.
  - `isDefault` (Boolean): Whether this is the default datasource. Default: `false`.
  - `jsonData` (Map): Additional JSON data specific to the datasource type.
  - `secureJsonData` (Map): Secure JSON data (e.g., passwords, tokens).

## 7. Best Practices for Configuration Management

1. **Version Control Everything:** All configuration files, schemas, and manifests must be stored in a version control system (e.g., Git). This enables tracking changes, rolling back, and peer review.
2. **Use Infrastructure as Code (IaC):** Avoid manual configuration via UI. Use tools like Terraform, Pulumi, or CloudFormation to define infrastructure declaratively.
3. **Keep Secrets Out of Source Control:** Never hardcode passwords, API keys, or tokens in configuration files. Use secret management tools like HashiCorp Vault, AWS Secrets Manager, or Kubernetes Secrets.
4. **Modularize Configurations:** Break down large configuration files into smaller, reusable modules or roles. This improves readability and maintainability.
5. **Implement CI/CD for Configurations:** Treat configuration changes like code changes. Run linters (e.g., `tflint`, `yamllint`), security scanners (e.g., `tfsec`, `checkov`), and automated tests before applying changes.
6. **Use Environment-Specific Variables:** Parameterize configurations to support multiple environments (e.g., dev, staging, prod) without duplicating code.
7. **Document Schemas and Defaults:** Maintain clear documentation for custom modules, roles, and templates, specifying required fields, default values, and examples.
8. **Enforce Least Privilege:** When configuring IAM roles, Kubernetes RBAC, or CI/CD permissions, always follow the principle of least privilege.
9. **Immutable Infrastructure:** Prefer replacing infrastructure over modifying it in place. This reduces configuration drift and ensures consistency.
10. **Regularly Audit Configurations:** Periodically review configurations for security vulnerabilities, deprecated features, and compliance with organizational standards.

## 8. Conclusion
Understanding and mastering the configuration schemas of DevOps tools is crucial for building robust, scalable, and secure systems. By adhering to the schemas and best practices outlined in this guide, DevOps engineers can ensure that their infrastructure, pipelines, and monitoring systems are configured optimally and consistently across all environments. Continuous learning and adaptation to evolving schemas are key to maintaining a cutting-edge DevOps practice.