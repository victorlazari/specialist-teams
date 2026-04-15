# NemoClaw Specialist

> **Role:** NVIDIA NemoClaw Platform Expert — OpenShell Integration, Blueprint System, and Enterprise Deployment
> **Official Sources:** [GitHub Repository](https://github.com/NVIDIA/NemoClaw) | [NVIDIA Documentation](https://docs.nvidia.com/nemoclaw/latest/)

---

## 1. Introduction and Overview

NVIDIA NemoClaw is an open-source reference stack (Apache 2.0 license, 19.3k+ GitHub stars) that simplifies running OpenClaw inside NVIDIA OpenShell with managed inference, enterprise-grade security, and comprehensive deployment support. Released as alpha software on March 16, 2026, NemoClaw represents NVIDIA's commitment to making AI assistant deployment accessible while maintaining the security and performance standards expected in enterprise environments.

NemoClaw is part of the broader NVIDIA Agent Toolkit ecosystem and is written primarily in TypeScript (70.4%), Shell (25.8%), and Python (2.4%). It adds a guided onboarding experience, a hardened blueprint system, state management, OpenShell-managed channel messaging, routed inference, and layered protection on top of the OpenClaw gateway.

The fundamental value proposition of NemoClaw is that it transforms the process of deploying a production-ready AI assistant from a complex, multi-step manual procedure into a streamlined, opinionated workflow that follows NVIDIA's best practices for security, performance, and reliability.

---

## 2. System Requirements and Prerequisites

Before deploying NemoClaw, specialists must ensure the target environment meets the minimum hardware and software requirements.

### 2.1 Hardware Requirements

| Resource | Minimum | Recommended | Notes |
|---|---|---|---|
| **CPU** | 4 vCPU | 8+ vCPU | More cores improve concurrent request handling |
| **RAM** | 8 GB | 16+ GB | Memory scales with concurrent sessions |
| **Disk** | 20 GB | 50+ GB | Includes Docker images and snapshot storage |
| **GPU** | Not required | NVIDIA GPU | Required for local inference with NeMo models |

### 2.2 Software Requirements

| Software | Required Version | Notes |
|---|---|---|
| **Node.js** | 22.16+ | NemoClaw requires a slightly newer Node.js than base OpenClaw |
| **npm** | 10+ | Comes with Node.js 22.16+ |
| **Docker** | Latest stable | Required for containerized deployment |
| **Git** | Latest stable | For cloning and updates |

### 2.3 Supported Platforms

NemoClaw supports multiple deployment platforms, each with specific considerations.

**Linux + Docker** is the primary and most thoroughly tested platform. Any modern Linux distribution with Docker Engine installed is supported. This is the recommended platform for production deployments.

**macOS Apple Silicon + Colima/Docker Desktop** provides development and testing support for macOS users with M-series chips. Colima is recommended over Docker Desktop for better performance and resource management. Note that GPU acceleration is not available on macOS.

**DGX Spark** is NVIDIA's dedicated AI workstation platform. NemoClaw includes optimized configurations for DGX Spark that leverage the platform's GPU resources for local inference.

**Windows WSL2** enables NemoClaw deployment on Windows through the Windows Subsystem for Linux 2. This requires WSL2 with a Linux distribution (Ubuntu recommended) and Docker Desktop with WSL2 backend integration.

---

## 3. Architecture and Core Components

NemoClaw's architecture layers on top of OpenClaw, adding enterprise-grade capabilities while preserving the flexibility of the underlying gateway.

### 3.1 Architectural Layers

| Layer | Component | Description |
|---|---|---|
| **User Interface** | Chat Platforms | WhatsApp, Telegram, Slack, etc. (inherited from OpenClaw) |
| **Gateway** | OpenClaw Core | Message routing, channel adapters, agent runtime |
| **Orchestration** | NemoClaw Blueprint | Hardened configuration, policy enforcement, state management |
| **Runtime** | NVIDIA OpenShell | Managed execution environment with security boundaries |
| **Inference** | Routed Inference | Intelligent routing to NVIDIA API, local models, or third-party providers |
| **Security** | Layered Protection | Tier-based policies, sandbox isolation, staleness detection |
| **State** | Snapshot System | Persistent state management with create/list/restore capabilities |

### 3.2 OpenShell Runtime

NVIDIA OpenShell provides the managed execution environment for NemoClaw. It handles process isolation, resource allocation, and security boundary enforcement. The OpenShell runtime ensures that AI agent processes are sandboxed from the host system and from each other, preventing unauthorized access to system resources.

### 3.3 Blueprint System

The blueprint system is NemoClaw's configuration management layer. A blueprint defines the complete specification for a NemoClaw deployment, including agent configurations, security policies, channel settings, inference routing rules, and resource limits. Blueprints are versioned and can be shared across deployments, enabling consistent configuration management across development, staging, and production environments.

> **Key Concept:** A NemoClaw blueprint is a declarative specification that defines the entire state of a deployment. Blueprints are hardened by default, meaning they include security best practices and sensible defaults that can be customized but not accidentally weakened.

### 3.4 Routed Inference

NemoClaw's routed inference system intelligently directs AI model requests to the most appropriate inference provider based on configurable rules. Requests can be routed to NVIDIA's API endpoints for cloud-based inference, to local GPU-accelerated models running on the same machine or cluster, or to third-party providers (OpenAI, Anthropic, etc.) as fallbacks. The routing system considers factors such as model availability, latency requirements, cost constraints, and data privacy policies when making routing decisions.

---

## 4. Installation and Onboarding

### 4.1 Quick Installation

NemoClaw provides a single-command installation script that handles all dependencies and configuration.

```bash
# Download and run the NemoClaw installer
curl -fsSL https://www.nvidia.com/nemoclaw.sh | bash
```

This script performs the following actions. It verifies system requirements (Node.js version, Docker availability, hardware resources). It downloads the NemoClaw package and its dependencies. It runs the interactive onboarding wizard. It configures the OpenShell runtime environment. It creates the initial blueprint with hardened defaults.

### 4.2 Interactive Onboarding

The `nemoclaw onboard` command launches an interactive wizard that guides specialists through the initial configuration process.

```bash
nemoclaw onboard
```

The onboarding wizard covers the following steps. It prompts for the deployment name and environment (development, staging, production). It configures channel connections (API keys, tokens, webhook URLs). It sets up inference routing (NVIDIA API key, local model paths, third-party provider keys). It defines security policies (access control, content policies, rate limits). It creates the initial snapshot for state recovery.

### 4.3 Post-Installation Verification

After installation, verify the deployment is functioning correctly:

```bash
# Check NemoClaw status
nemoclaw status

# Verify all services are running
nemoclaw health

# Test channel connectivity
nemoclaw test channels
```

---

## 5. CLI Reference

NemoClaw provides a comprehensive CLI for managing all aspects of the deployment.

| Command | Description | Example |
|---|---|---|
| `nemoclaw onboard` | Interactive setup wizard | `nemoclaw onboard` |
| `nemoclaw status` | Display deployment status | `nemoclaw status` |
| `nemoclaw health` | Run health checks | `nemoclaw health` |
| `nemoclaw start` | Start all services | `nemoclaw start` |
| `nemoclaw stop` | Stop all services | `nemoclaw stop` |
| `nemoclaw restart` | Restart all services | `nemoclaw restart` |
| `nemoclaw snapshot create` | Create a state snapshot | `nemoclaw snapshot create --name "v1.0"` |
| `nemoclaw snapshot list` | List all snapshots | `nemoclaw snapshot list` |
| `nemoclaw snapshot restore` | Restore from snapshot | `nemoclaw snapshot restore --name "v1.0"` |
| `nemoclaw blueprint show` | Display current blueprint | `nemoclaw blueprint show` |
| `nemoclaw blueprint validate` | Validate blueprint syntax | `nemoclaw blueprint validate` |
| `nemoclaw logs` | View service logs | `nemoclaw logs --follow` |
| `nemoclaw update` | Update NemoClaw | `nemoclaw update` |

---

## 6. Security Architecture

NemoClaw implements a multi-layered security architecture that provides defense in depth.

### 6.1 Tier-Based Policy Selector

The policy selector allows administrators to define security tiers that control access to different capabilities. Each tier specifies which channels, agents, tools, and inference providers are available.

| Tier | Access Level | Typical Use |
|---|---|---|
| **Public** | Read-only, limited responses | Public-facing chatbots with restricted capabilities |
| **Standard** | Full chat, basic tools | Internal team members with standard AI access |
| **Privileged** | All tools, code execution | Developers and engineers with full agent capabilities |
| **Admin** | Full access + configuration | System administrators with deployment management |

### 6.2 Sandbox Version Staleness Detection

NemoClaw monitors the versions of all components (OpenClaw, OpenShell, Node.js, Docker images) and alerts administrators when any component becomes stale. Stale components may contain known vulnerabilities, and the staleness detection system provides actionable recommendations for updates.

### 6.3 Layered Protection

The security architecture implements protection at multiple layers. Network-level protection includes TLS encryption, firewall rules, and VPC isolation. Application-level protection includes authentication, authorization, and rate limiting. Agent-level protection includes tool access control, content policies, and output filtering. Data-level protection includes encryption at rest, encryption in transit, and audit logging.

---

## 7. State Management and Snapshots

NemoClaw's snapshot system provides robust state management capabilities that enable backup, recovery, and migration of deployments.

### 7.1 Creating Snapshots

Snapshots capture the complete state of a NemoClaw deployment, including the blueprint configuration, session data, plugin state, and system settings.

```bash
# Create a named snapshot
nemoclaw snapshot create --name "pre-upgrade-backup"

# Create a snapshot with description
nemoclaw snapshot create --name "v2.0-release" --description "Production release v2.0"
```

### 7.2 Restoring from Snapshots

```bash
# List available snapshots
nemoclaw snapshot list

# Restore a specific snapshot
nemoclaw snapshot restore --name "pre-upgrade-backup"
```

### 7.3 Snapshot Best Practices

Specialists should create snapshots before any significant configuration changes, before upgrading NemoClaw or its dependencies, on a regular schedule (daily for production environments), and before and after security policy changes. Snapshots should be stored in a durable location (e.g., S3, NFS) and tested regularly to ensure they can be restored successfully.

---

## 8. Kubernetes and Docker Deployment

### 8.1 Docker Deployment

NemoClaw includes a Dockerfile for containerized deployment.

```bash
# Build the NemoClaw Docker image
docker build -t nemoclaw:latest .

# Run NemoClaw in Docker
docker run -d \
  --name nemoclaw \
  -p 3000:3000 \
  -v nemoclaw-data:/data \
  -e NVIDIA_API_KEY=your-key \
  nemoclaw:latest
```

### 8.2 Kubernetes Deployment

NemoClaw provides sample Kubernetes manifests for cluster deployment.

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nemoclaw
  namespace: ai-agents
spec:
  replicas: 2
  selector:
    matchLabels:
      app: nemoclaw
  template:
    metadata:
      labels:
        app: nemoclaw
    spec:
      containers:
      - name: nemoclaw
        image: nvcr.io/nvidia/nemoclaw:latest
        ports:
        - containerPort: 3000
        env:
        - name: NVIDIA_API_KEY
          valueFrom:
            secretKeyRef:
              name: nemoclaw-secrets
              key: nvidia-api-key
        resources:
          requests:
            cpu: "2"
            memory: "4Gi"
          limits:
            cpu: "4"
            memory: "8Gi"
        volumeMounts:
        - name: data
          mountPath: /data
      volumes:
      - name: data
        persistentVolumeClaim:
          claimName: nemoclaw-pvc
```

---

## 9. Troubleshooting

| Issue | Cause | Resolution |
|---|---|---|
| Installation script fails | Missing prerequisites | Run `nemoclaw health` to identify missing dependencies |
| Docker containers not starting | Port conflicts or resource limits | Check `docker logs nemoclaw` and verify port availability |
| Inference routing errors | Invalid API keys or unreachable endpoints | Verify provider configuration in blueprint |
| Snapshot restore fails | Corrupted snapshot or version mismatch | Try an older snapshot; check NemoClaw version compatibility |
| Staleness warnings | Outdated components | Run `nemoclaw update` to update all components |
| OpenShell connection errors | Docker daemon not running | Start Docker daemon and restart NemoClaw |

---

## 10. Advanced Topics

For advanced blueprint customization, policy tier configuration, snapshot management strategies, OpenShell gateway internals, DGX Spark deployment, and enterprise scaling patterns, refer to the companion document **[02-nemoclaw-advanced.md](./02-nemoclaw-advanced.md)**.

---

## References

1. NVIDIA NemoClaw GitHub Repository — https://github.com/NVIDIA/NemoClaw
2. NVIDIA NemoClaw Documentation — https://docs.nvidia.com/nemoclaw/latest/
3. NVIDIA OpenShell Documentation — https://docs.nvidia.com/openshell/
4. NVIDIA Agent Toolkit — https://developer.nvidia.com/agent-toolkit
5. OpenClaw Documentation — https://docs.openclaw.ai/
