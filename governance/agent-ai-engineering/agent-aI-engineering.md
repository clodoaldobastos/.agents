---
name: agent-ai-engineering
version: 1.0.0
description: >
  Enterprise AI Platform v2 - Proposta de arquitetura para plataforma
  corporativa de IA com agentes autônomos, MCP, RAG, AI Ops
  e Platform Engineering.
owner: platform-engineering
maturity_level: enterprise
tags:
  - ai-platform
  - architecture
  - platform-engineering
  - ai-governance
  - mcp
capabilities:
  - platform-architecture-design
  - ai-platform-proposal
  - directory-structure-design
tools:
  - filesystem
  - bash
---

Eu consolidaria tudo em uma versão Enterprise AI Platform v2, incorporando:

Agentes Autônomos
MCP
RAG
AI Ops
FinOps
Platform Engineering
Observabilidade
Governança
Segurança
Runtime de execução
Memory Layer
Context Engineering
Self-Service Platform


.opencode/
│
├── agents/
│   ├── orchestrator-agent/
│   ├── kubernetes-agent/
│   ├── terraform-agent/
│   ├── sre-agent/
│   ├── security-agent/
│   ├── gitlab-agent/
│   ├── keycloak-agent/
│   ├── finops-agent/
│   ├── observability-agent/
│   ├── documentation-agent/
│   ├── compliance-agent/
│   └── platform-agent/
│
├── skills/
│   ├── kubernetes/
│   ├── terraform/
│   ├── ansible/
│   ├── helm/
│   ├── gitlab/
│   ├── keycloak/
│   ├── azure/
│   ├── aws/
│   ├── security/
│   ├── prometheus/
│   ├── grafana/
│   ├── loki/
│   ├── opentelemetry/
│   └── finops/
│
├── workflows/
│   ├── incident-response/
│   ├── root-cause-analysis/
│   ├── cluster-provisioning/
│   ├── application-deployment/
│   ├── user-onboarding/
│   ├── compliance-audit/
│   ├── disaster-recovery/
│   ├── cost-optimization/
│   └── security-investigation/
│
├── chains/
│   ├── deployment-chain/
│   ├── troubleshooting-chain/
│   ├── sre-chain/
│   ├── security-chain/
│   ├── governance-chain/
│   └── aiops-chain/
│
├── rag/
│   ├── confluence/
│   ├── wiki/
│   ├── runbooks/
│   ├── playbooks/
│   ├── architecture/
│   ├── gitlab-docs/
│   ├── terraform-modules/
│   ├── helm-charts/
│   ├── policies/
│   └── standards/
│
├── memory/
│   ├── short-term/
│   ├── long-term/
│   ├── semantic/
│   ├── episodic/
│   └── conversation/
│
├── context/
│   ├── cluster-context/
│   ├── project-context/
│   ├── environment-context/
│   ├── user-context/
│   ├── tenant-context/
│   └── business-context/
│
├── mcp/
│   ├── gitlab/
│   ├── github/
│   ├── kubernetes/
│   ├── prometheus/
│   ├── grafana/
│   ├── loki/
│   ├── tempo/
│   ├── keycloak/
│   ├── jira/
│   ├── slack/
│   ├── teams/
│   ├── azure/
│   ├── aws/
│   ├── gcp/
│   ├── vault/
│   └── databases/
│
├── adapters/
│   ├── rest/
│   ├── grpc/
│   ├── webhook/
│   ├── kafka/
│   ├── rabbitmq/
│   ├── eventhub/
│   └── custom/
│
├── runtime/
│   ├── execution-engine/
│   ├── containers/
│   ├── sandboxes/
│   ├── queues/
│   ├── schedulers/
│   └── workers/
│
├── aiops/
│   ├── anomaly-detection/
│   ├── root-cause-analysis/
│   ├── auto-remediation/
│   ├── forecasting/
│   ├── alert-correlation/
│   └── incident-intelligence/
│
├── finops/
│   ├── azure-costs/
│   ├── aws-costs/
│   ├── budgets/
│   ├── chargeback/
│   ├── showback/
│   ├── optimization/
│   └── recommendations/
│
├── platform-engineering/
│   ├── developer-portal/
│   ├── self-service/
│   ├── golden-paths/
│   ├── scorecards/
│   ├── software-templates/
│   ├── idp/
│   └── internal-products/
│
├── observability/
│   ├── prometheus/
│   ├── grafana/
│   ├── loki/
│   ├── tempo/
│   ├── opentelemetry/
│   ├── dashboards/
│   └── alerts/
│
├── telemetry/
│   ├── metrics/
│   ├── logs/
│   ├── traces/
│   ├── events/
│   └── profiling/
│
├── governance/
│   ├── standards/
│   ├── compliance/
│   ├── approvals/
│   ├── audits/
│   ├── risk-management/
│   └── architecture-review/
│
├── policies/
│   ├── access/
│   ├── execution/
│   ├── llm/
│   ├── security/
│   ├── compliance/
│   └── data-protection/
│
├── auth/
│   ├── keycloak/
│   ├── azure-ad/
│   ├── workload-identity/
│   ├── service-accounts/
│   ├── rbac/
│   └── sso/
│
├── registry/
│   ├── agents/
│   ├── skills/
│   ├── prompts/
│   ├── workflows/
│   ├── templates/
│   └── versions/
│
├── prompts/
│   ├── system/
│   ├── agents/
│   ├── workflows/
│   ├── governance/
│   └── templates/
│
├── templates/
│   ├── agent-template/
│   ├── skill-template/
│   ├── workflow-template/
│   ├── chain-template/
│   └── project-template/
│
├── tools/
│   ├── kubectl/
│   ├── helm/
│   ├── terraform/
│   ├── ansible/
│   ├── azcli/
│   ├── awscli/
│   ├── k9s/
│   ├── jq/
│   ├── yq/
│   └── scripts/
│
├── hooks/
│   ├── pre-execution/
│   ├── post-execution/
│   ├── approval-hooks/
│   ├── audit-hooks/
│   └── notification-hooks/
│
├── docs/
│   ├── adr/
│   ├── architecture/
│   ├── onboarding/
│   ├── runbooks/
│   ├── playbooks/
│   └── standards/
│
├── platform/
│   ├── aks/
│   ├── eks/
│   ├── gke/
│   ├── openshift/
│   ├── onprem/
│   └── hybrid/
│
├── rules/
│   ├── guardrails/
│   ├── validations/
│   ├── naming/
│   ├── security/
│   └── best-practices/
│
├── testing/
│   ├── agent-tests/
│   ├── workflow-tests/
│   ├── skill-tests/
│   ├── policy-tests/
│   └── integration-tests/
│
├── security/
│   ├── scanning/
│   ├── secrets-detection/
│   ├── vulnerability-management/
│   ├── supply-chain/
│   └── compliance/
│
└── .secrets/
    ├── vault/
    ├── keyvault/
    ├── certificates/
    ├── credentials/
    └── encryption/


┌─────────────────────────────────────────┐
│               Usuários                  │
└─────────────────────────────────────────┘
                    │
                    ▼
┌─────────────────────────────────────────┐
│          Agent Orchestrator             │
└─────────────────────────────────────────┘
                    │
     ┌──────────────┼──────────────┐
     ▼              ▼              ▼

  RAG Layer    Memory Layer    Context Layer

     ▼              ▼              ▼

┌─────────────────────────────────────────┐
│         Specialized Agents              │
├─────────────────────────────────────────┤
│ K8s │ Terraform │ SRE │ Security │ etc │
└─────────────────────────────────────────┘
                    │
                    ▼
┌─────────────────────────────────────────┐
│             MCP Layer                   │
├─────────────────────────────────────────┤
│ GitLab │ K8s │ Azure │ Jira │ Grafana  │
└─────────────────────────────────────────┘
                    │
                    ▼
┌─────────────────────────────────────────┐
│      Workflow + Runtime Engine          │
└─────────────────────────────────────────┘
                    │
                    ▼
┌─────────────────────────────────────────┐
│     AI Ops + Platform Engineering       │
└─────────────────────────────────────────┘
                    │
                    ▼
┌─────────────────────────────────────────┐
│ Observability + Governance + Security   │
└─────────────────────────────────────────┘

Essa estrutura já fica próxima do que equipes maduras de Platform Engineering e AI Engineering implementam para operar agentes corporativos em ambientes como AKS, GitLab, Terraform, Keycloak, Prometheus, Grafana e OpenTelemetry.