# Specialist Teams Knowledge Base

This repository contains the ultimate reference guides for 50 different specialist domains. Each specialist now includes a comprehensive multi-file structure with 7 documents covering every aspect of the domain. When an AI agent needs deep expertise, architecture patterns, code examples, troubleshooting guidance, security audits, or configuration references in a specific domain, they should refer to the appropriate files in this repository.

## Usage Instructions for AI Agents

1. Identify the domain of your current task.
2. Locate the corresponding specialist files from the directory below.
3. Read the **main file** (e.g., `01-openclaw-specialist.md`) for core concepts, architecture, and standard workflows.
4. Read the **advanced file** (e.g., `01-openclaw-advanced.md`) for complex scenarios, scaling, security, and edge cases.
5. Read the **cli-reference** file for complete command-line tool documentation.
6. Read the **troubleshooting** file for error codes, diagnostics, and recovery strategies.
7. Read the **security-audit** file for security validation checklists and hardening.
8. Read the **config-schemas** file for all configuration files, fields, and defaults.
9. Read the **deep-dive** file for advanced architecture, performance tuning, and enterprise patterns.

## File Naming Convention

Each specialist follows this naming pattern:
- `{ID}-{name}-specialist.md` — Core concepts and standard workflows
- `{ID}-{name}-advanced.md` — Complex scenarios and scaling
- `{ID}-{name}-cli-reference.md` — CLI command reference
- `{ID}-{name}-troubleshooting.md` — Diagnostics and error recovery
- `{ID}-{name}-security-audit.md` — Security checklist and hardening
- `{ID}-{name}-config-schemas.md` — Configuration documentation
- `{ID}-{name}-deep-dive.md` — Advanced architecture and performance

## Available Specialists

| # | Domain | Main | Advanced | CLI Ref | Troubleshoot | Security | Config | Deep Dive |
|---|--------|------|----------|---------|--------------|----------|--------|-----------|
| 01 | OpenClaw | `01-openclaw-specialist.md` | `01-openclaw-advanced.md` | `01-openclaw-cli-reference.md` | `01-openclaw-troubleshooting.md` | `01-openclaw-security-audit.md` | `01-openclaw-config-schemas.md` | `01-openclaw-deep-dive.md` |
| 02 | NemoClaw | `02-nemoclaw-specialist.md` | `02-nemoclaw-advanced.md` | `02-nemoclaw-cli-reference.md` | `02-nemoclaw-troubleshooting.md` | `02-nemoclaw-security-audit.md` | `02-nemoclaw-config-schemas.md` | `02-nemoclaw-deep-dive.md` |
| 03 | Prompt Engineering | `03-prompt-specialist.md` | `03-prompt-advanced.md` | `03-prompt-cli-reference.md` | `03-prompt-troubleshooting.md` | `03-prompt-security-audit.md` | `03-prompt-config-schemas.md` | `03-prompt-deep-dive.md` |
| 04 | RAG | `04-rag-specialist.md` | `04-rag-advanced.md` | `04-rag-cli-reference.md` | `04-rag-troubleshooting.md` | `04-rag-security-audit.md` | `04-rag-config-schemas.md` | `04-rag-deep-dive.md` |
| 05 | AI Fundamentals | `05-ai-specialist.md` | `05-ai-advanced.md` | `05-ai-cli-reference.md` | `05-ai-troubleshooting.md` | `05-ai-security-audit.md` | `05-ai-config-schemas.md` | `05-ai-deep-dive.md` |
| 06 | Claude (Anthropic) | `06-claude-specialist.md` | `06-claude-advanced.md` | `06-claude-cli-reference.md` | `06-claude-troubleshooting.md` | `06-claude-security-audit.md` | `06-claude-config-schemas.md` | `06-claude-deep-dive.md` |
| 07 | Manus Platform | `07-manus-specialist.md` | `07-manus-advanced.md` | `07-manus-cli-reference.md` | `07-manus-troubleshooting.md` | `07-manus-security-audit.md` | `07-manus-config-schemas.md` | `07-manus-deep-dive.md` |
| 08 | Manus Workflows | `08-manus-specialist-2.md` | `08-manus-advanced-2.md` | `08-manus-workflows-cli-reference.md` | `08-manus-workflows-troubleshooting.md` | `08-manus-workflows-security-audit.md` | `08-manus-workflows-config-schemas.md` | `08-manus-workflows-deep-dive.md` |
| 09 | OpenAI | `09-openai-specialist.md` | `09-openai-advanced.md` | `09-openai-cli-reference.md` | `09-openai-troubleshooting.md` | `09-openai-security-audit.md` | `09-openai-config-schemas.md` | `09-openai-deep-dive.md` |
| 10 | Databases (PG/Mongo) | `10-database-specialist.md` | `10-database-advanced.md` | `10-database-cli-reference.md` | `10-database-troubleshooting.md` | `10-database-security-audit.md` | `10-database-config-schemas.md` | `10-database-deep-dive.md` |
| 11 | Messaging (RabbitMQ/DocDB) | `11-rabbitmq-documentdb-specialist.md` | `11-rabbitmq-documentdb-advanced.md` | `11-rabbitmq-documentdb-cli-reference.md` | `11-rabbitmq-documentdb-troubleshooting.md` | `11-rabbitmq-documentdb-security-audit.md` | `11-rabbitmq-documentdb-config-schemas.md` | `11-rabbitmq-documentdb-deep-dive.md` |
| 12 | Caching (Valkey/Redis) | `12-valkey-redis-specialist.md` | `12-valkey-redis-advanced.md` | `12-valkey-redis-cli-reference.md` | `12-valkey-redis-troubleshooting.md` | `12-valkey-redis-security-audit.md` | `12-valkey-redis-config-schemas.md` | `12-valkey-redis-deep-dive.md` |
| 13 | Go (Golang) | `13-go-specialist.md` | `13-go-advanced.md` | `13-go-cli-reference.md` | `13-go-troubleshooting.md` | `13-go-security-audit.md` | `13-go-config-schemas.md` | `13-go-deep-dive.md` |
| 14 | Frontend (React/Next) | `14-frontend-specialist.md` | `14-frontend-advanced.md` | `14-frontend-cli-reference.md` | `14-frontend-troubleshooting.md` | `14-frontend-security-audit.md` | `14-frontend-config-schemas.md` | `14-frontend-deep-dive.md` |
| 15 | DevOps (AWS/K8s/EKS) | `15-devops-specialist.md` | `15-devops-advanced.md` | `15-devops-cli-reference.md` | `15-devops-troubleshooting.md` | `15-devops-security-audit.md` | `15-devops-config-schemas.md` | `15-devops-deep-dive.md` |
| 16 | Lua | `16-lua-specialist.md` | `16-lua-advanced.md` | `16-lua-cli-reference.md` | `16-lua-troubleshooting.md` | `16-lua-security-audit.md` | `16-lua-config-schemas.md` | `16-lua-deep-dive.md` |
| 17 | Bash/Shell | `17-bash-specialist.md` | `17-bash-advanced.md` | `17-bash-cli-reference.md` | `17-bash-troubleshooting.md` | `17-bash-security-audit.md` | `17-bash-config-schemas.md` | `17-bash-deep-dive.md` |
| 18 | Ticket System Supreme | `18-ticket-supreme-specialist.md` | `18-ticket-supreme-advanced.md` | `18-ticket-supreme-cli-reference.md` | `18-ticket-supreme-troubleshooting.md` | `18-ticket-supreme-security-audit.md` | `18-ticket-supreme-config-schemas.md` | `18-ticket-supreme-deep-dive.md` |
| 19 | On-Call Master Supreme | `19-oncall-master-supreme-specialist.md` | `19-oncall-master-supreme-advanced.md` | `19-oncall-master-supreme-cli-reference.md` | `19-oncall-master-supreme-troubleshooting.md` | `19-oncall-master-supreme-security-audit.md` | `19-oncall-master-supreme-config-schemas.md` | `19-oncall-master-supreme-deep-dive.md` |
| 20 | Jira JSM Alerts & On-Call | `20-jira-jsm-oncall-specialist.md` | `20-jira-jsm-oncall-advanced.md` | `20-jira-jsm-oncall-cli-reference.md` | `20-jira-jsm-oncall-troubleshooting.md` | `20-jira-jsm-oncall-security-audit.md` | `20-jira-jsm-oncall-config-schemas.md` | `20-jira-jsm-oncall-deep-dive.md` |
| 21 | VoIP On-Call Services | `21-voip-oncall-specialist.md` | `21-voip-oncall-advanced.md` | `21-voip-oncall-cli-reference.md` | `21-voip-oncall-troubleshooting.md` | `21-voip-oncall-security-audit.md` | `21-voip-oncall-config-schemas.md` | `21-voip-oncall-deep-dive.md` |
| 22 | Ticket System Reports | `22-ticket-reports-specialist.md` | `22-ticket-reports-advanced.md` | `22-ticket-reports-cli-reference.md` | `22-ticket-reports-troubleshooting.md` | `22-ticket-reports-security-audit.md` | `22-ticket-reports-config-schemas.md` | `22-ticket-reports-deep-dive.md` |
| 23 | Jira Status & Workflows | `23-jira-status-workflows-specialist.md` | `23-jira-status-workflows-advanced.md` | `23-jira-status-workflows-cli-reference.md` | `23-jira-status-workflows-troubleshooting.md` | `23-jira-status-workflows-security-audit.md` | `23-jira-status-workflows-config-schemas.md` | `23-jira-status-workflows-deep-dive.md` |
| 24 | SeaweedFS | `24-seaweedfs-specialist.md` | `24-seaweedfs-advanced.md` | `24-seaweedfs-cli-reference.md` | `24-seaweedfs-troubleshooting.md` | `24-seaweedfs-security-audit.md` | `24-seaweedfs-config-schemas.md` | `24-seaweedfs-deep-dive.md` |
| 25 | Speedtest (Ookla) | `25-speedtest-specialist.md` | `25-speedtest-advanced.md` | `25-speedtest-cli-reference.md` | `25-speedtest-troubleshooting.md` | `25-speedtest-security-audit.md` | `25-speedtest-config-schemas.md` | `25-speedtest-deep-dive.md` |
| 26 | Frontend Menu Design | `26-frontend-menu-design-specialist.md` | `26-frontend-menu-design-advanced.md` | `26-frontend-menu-design-cli-reference.md` | `26-frontend-menu-design-troubleshooting.md` | `26-frontend-menu-design-security-audit.md` | `26-frontend-menu-design-config-schemas.md` | `26-frontend-menu-design-deep-dive.md` |
| 27 | Web Tester Supreme | `27-web-tester-supreme-specialist.md` | `27-web-tester-supreme-advanced.md` | `27-web-tester-supreme-cli-reference.md` | `27-web-tester-supreme-troubleshooting.md` | `27-web-tester-supreme-security-audit.md` | `27-web-tester-supreme-config-schemas.md` | `27-web-tester-supreme-deep-dive.md` |
| 28 | Playwright E2E | `28-playwright-specialist.md` | `28-playwright-advanced.md` | `28-playwright-cli-reference.md` | `28-playwright-troubleshooting.md` | `28-playwright-security-audit.md` | `28-playwright-config-schemas.md` | `28-playwright-deep-dive.md` |
| 29 | Vitest Unit Testing | `29-vitest-specialist.md` | `29-vitest-advanced.md` | `29-vitest-cli-reference.md` | `29-vitest-troubleshooting.md` | `29-vitest-security-audit.md` | `29-vitest-config-schemas.md` | `29-vitest-deep-dive.md` |
| 30 | Accessibility Testing | `30-accessibility-testing-specialist.md` | `30-accessibility-testing-advanced.md` | `30-accessibility-testing-cli-reference.md` | `30-accessibility-testing-troubleshooting.md` | `30-accessibility-testing-security-audit.md` | `30-accessibility-testing-config-schemas.md` | `30-accessibility-testing-deep-dive.md` |
| 31 | Jira Field Schemas | `31-jira-field-schemas-specialist.md` | `31-jira-field-schemas-advanced.md` | `31-jira-field-schemas-cli-reference.md` | `31-jira-field-schemas-troubleshooting.md` | `31-jira-field-schemas-security-audit.md` | `31-jira-field-schemas-config-schemas.md` | `31-jira-field-schemas-deep-dive.md` |
| 32 | Roles & Permissions | `32-roles-permissions-specialist.md` | `32-roles-permissions-advanced.md` | `32-roles-permissions-cli-reference.md` | `32-roles-permissions-troubleshooting.md` | `32-roles-permissions-security-audit.md` | `32-roles-permissions-config-schemas.md` | `32-roles-permissions-deep-dive.md` |
| 33 | Dockerfile Mastery | `33-dockerfile-specialist.md` | `33-dockerfile-advanced.md` | `33-dockerfile-cli-reference.md` | `33-dockerfile-troubleshooting.md` | `33-dockerfile-security-audit.md` | `33-dockerfile-config-schemas.md` | `33-dockerfile-deep-dive.md` |
| 34 | Bot (OpenClaw/NemoClaw/OpenShell) | `34-bot-specialist.md` | `34-bot-advanced.md` | `34-bot-cli-reference.md` | `34-bot-troubleshooting.md` | `34-bot-security-audit.md` | `34-bot-config-schemas.md` | `34-bot-deep-dive.md` |
| 35 | Spanish Teacher | `35-spanish-teacher-specialist.md` | `35-spanish-teacher-advanced.md` | `35-spanish-teacher-cli-reference.md` | `35-spanish-teacher-troubleshooting.md` | `35-spanish-teacher-security-audit.md` | `35-spanish-teacher-config-schemas.md` | `35-spanish-teacher-deep-dive.md` |
| 36 | French Teacher | `36-french-teacher-specialist.md` | `36-french-teacher-advanced.md` | `36-french-teacher-cli-reference.md` | `36-french-teacher-troubleshooting.md` | `36-french-teacher-security-audit.md` | `36-french-teacher-config-schemas.md` | `36-french-teacher-deep-dive.md` |
| 37 | Kubernetes & EKS | `37-k8s-eks-specialist.md` | `37-k8s-eks-advanced.md` | `37-k8s-eks-cli-reference.md` | `37-k8s-eks-troubleshooting.md` | `37-k8s-eks-security-audit.md` | `37-k8s-eks-config-schemas.md` | `37-k8s-eks-deep-dive.md` |
| 38 | PostgreSQL 15+ | `38-postgres-15-specialist.md` | `38-postgres-15-advanced.md` | `38-postgres-15-cli-reference.md` | `38-postgres-15-troubleshooting.md` | `38-postgres-15-security-audit.md` | `38-postgres-15-config-schemas.md` | `38-postgres-15-deep-dive.md` |
| 39 | RabbitMQ | `39-rabbitmq-specialist.md` | `39-rabbitmq-advanced.md` | `39-rabbitmq-cli-reference.md` | `39-rabbitmq-troubleshooting.md` | `39-rabbitmq-security-audit.md` | `39-rabbitmq-config-schemas.md` | `39-rabbitmq-deep-dive.md` |
| 40 | Redis & Valkey | `40-redis-valkey-specialist.md` | `40-redis-valkey-advanced.md` | `40-redis-valkey-cli-reference.md` | `40-redis-valkey-troubleshooting.md` | `40-redis-valkey-security-audit.md` | `40-redis-valkey-config-schemas.md` | `40-redis-valkey-deep-dive.md` |
| 41 | MongoDB | `41-mongodb-specialist.md` | `41-mongodb-advanced.md` | `41-mongodb-cli-reference.md` | `41-mongodb-troubleshooting.md` | `41-mongodb-security-audit.md` | `41-mongodb-config-schemas.md` | `41-mongodb-deep-dive.md` |
| 42 | Go & Lua | `42-go-lua-specialist.md` | `42-go-lua-advanced.md` | `42-go-lua-cli-reference.md` | `42-go-lua-troubleshooting.md` | `42-go-lua-security-audit.md` | `42-go-lua-config-schemas.md` | `42-go-lua-deep-dive.md` |
| 43 | Lerian Helm Deployments | `43-lerian-helm-specialist.md` | `43-lerian-helm-advanced.md` | `43-lerian-helm-cli-reference.md` | `43-lerian-helm-troubleshooting.md` | `43-lerian-helm-security-audit.md` | `43-lerian-helm-config-schemas.md` | `43-lerian-helm-deep-dive.md` |
| 44 | SQL & Database Partitioning | `44-sql-partitioning-specialist.md` | `44-sql-partitioning-advanced.md` | `44-sql-partitioning-cli-reference.md` | `44-sql-partitioning-troubleshooting.md` | `44-sql-partitioning-security-audit.md` | `44-sql-partitioning-config-schemas.md` | `44-sql-partitioning-deep-dive.md` |
| 45 | Tech Support Operations | `45-tech-support-ops-specialist.md` | `45-tech-support-ops-advanced.md` | `45-tech-support-ops-cli-reference.md` | `45-tech-support-ops-troubleshooting.md` | `45-tech-support-ops-security-audit.md` | `45-tech-support-ops-config-schemas.md` | `45-tech-support-ops-deep-dive.md` |
| 46 | Wiki.js | `46-wikijs-specialist.md` | `46-wikijs-advanced.md` | `46-wikijs-cli-reference.md` | `46-wikijs-troubleshooting.md` | `46-wikijs-security-audit.md` | `46-wikijs-config-schemas.md` | `46-wikijs-deep-dive.md` |
| 47 | Hermes Agent (NousResearch) | `47-hermes-agent-specialist.md` | `47-hermes-agent-advanced.md` | `47-hermes-agent-cli-reference.md` | `47-hermes-agent-troubleshooting.md` | `47-hermes-agent-security-audit.md` | `47-hermes-agent-config-schemas.md` | `47-hermes-agent-deep-dive.md` |
| 48 | OpenTelemetry Collector | `48-otel-collector-specialist.md` | `48-otel-collector-advanced.md` | `48-otel-collector-cli-reference.md` | `48-otel-collector-troubleshooting.md` | `48-otel-collector-security-audit.md` | `48-otel-collector-config-schemas.md` | `48-otel-collector-deep-dive.md` |
| 49 | Docker Super Specialist | `49-docker-specialist.md` | `49-docker-advanced.md` | `49-docker-cli-reference.md` | `49-docker-troubleshooting.md` | `49-docker-security-audit.md` | `49-docker-config-schemas.md` | `49-docker-deep-dive.md` |
| 50 | Slack Master Specialist | `50-slack-specialist.md` | `50-slack-advanced.md` | `50-slack-cli-reference.md` | `50-slack-troubleshooting.md` | `50-slack-security-audit.md` | `50-slack-config-schemas.md` | `50-slack-deep-dive.md` |

## File Structure

All markdown files are structured with one main file per skill and several complementary child files. The main file contains instructions to refer to child files for additional details when necessary.

## Quick Reference by Task Type

| Task Type | Files to Read |
|-----------|---------------|
| **Setting up a new system** | `*-specialist.md` + `*-config-schemas.md` |
| **Debugging an issue** | `*-troubleshooting.md` + `*-cli-reference.md` |
| **Security review** | `*-security-audit.md` |
| **Performance optimization** | `*-deep-dive.md` + `*-advanced.md` |
| **Learning the domain** | `*-specialist.md` + `*-advanced.md` |
| **Day-to-day operations** | `*-cli-reference.md` + `*-troubleshooting.md` |
