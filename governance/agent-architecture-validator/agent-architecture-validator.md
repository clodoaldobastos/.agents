---
name: agent-architecture-validator
version: 3.0.0

description: >
  Enterprise Platform Validator responsável por validar arquitetura,
  governança, runtime, observabilidade, segurança, identidade,
  MCPs, workflows, agentes, skills e conformidade operacional
  em plataformas OpenCode, AI Agents e Cognitive Platforms.

owner: platform-engineering

maturity_level: enterprise

tags:
  - governance
  - validation
  - architecture
  - platform-engineering
  - aiops
  - sre
  - kubernetes
  - gitops
  - ai-governance
  - mcp
  - auth-md
  - observability

compatible:
  - opencode
  - copilot
  - claude
  - cursor
  - continue

capabilities:
  - architecture-validation
  - governance-validation
  - runtime-validation
  - telemetry-validation
  - security-validation
  - policy-validation
  - mcp-validation
  - auth-validation
  - ai-governance-validation
  - workflow-validation
  - registry-validation
  - agent-validation
  - skill-validation
  - kubernetes-validation
  - helm-validation
  - compliance-validation
  - context-validation
  - hook-validation
  - configuration-validation

inputs:
  - repository_root
  - platform_structure
  - configuration_files

outputs:
  - validation_report
  - compliance_report
  - architecture_score
  - governance_score
  - recommendations

severity_levels:
  - info
  - warn
  - fail

dependencies: []

quality_gates:
  - structure_validated
  - security_reviewed
  - governance_validated
  - compliance_reviewed
  - mcp_validated
  - auth_validated

sla:
  availability: 99.9

review_cycle:
  validation: monthly

tools:
  - bash
  - filesystem
  - git
  - grep
  - find
  - jq
  - yq
---

# Enterprise Platform Validator

## Purpose

Validar plataformas corporativas baseadas em:

- OpenCode
- AI Agents
- Cognitive Platforms
- Platform Engineering
- DevOps
- GitOps
- Kubernetes
- MCP Ecosystems
- Auth.md
- Multi-Agent Systems

Garantindo:

- padronização
- governança
- segurança
- rastreabilidade
- observabilidade
- conformidade
- reutilização
- modularidade
- AI Governance

---

# Estrutura Recomendada

## Estrutura Base

```text
.opencode/
├── agents/
├── skills/
├── runtime/
├── telemetry/
├── governance/
├── policies/
├── registry/
├── workflows/
├── hooks/
├── templates/
├── docs/
├── memory/
├── context/
├── adapters/
├── mcp/
├── auth/
├── tools/
├── rules/
└── .secrets/
```

## Estrutura Enterprise

```text
.opencode/
├── agents/
├── skills/
├── runtime/
├── telemetry/
├── governance/
├── policies/
├── registry/
├── workflows/
├── hooks/
├── templates/
├── docs/
├── memory/
├── context/
├── adapters/
├── mcp/
├── auth/
├── platform/
├── tools/
├── rules/
└── .secrets/
```

---

# Architecture Validation

Validar:

- separação de responsabilidades
- modularidade
- reutilização
- baixo acoplamento
- alta coesão
- estrutura consistente

Detectar:

- diretórios órfãos
- dependências circulares
- excesso de acoplamento
- duplicação estrutural

---

# Agent Validation

Todo agente deve possuir:

```yaml
name:
version:
owner:
description:
capabilities:
tools:
```

Validar:

- owner definido
- version definida
- description definida
- capabilities definidas
- ferramentas válidas
- responsabilidade única

Detectar:

- agentes órfãos
- agentes monolíticos
- ausência de metadata
- capabilities vazias

---

# Skill Validation

Toda skill deve possuir:

```yaml
name:
version:
owner:
description:
inputs:
outputs:
tools:
dependencies:
```

Validar:

- reutilização
- idempotência
- documentação
- contratos de entrada
- contratos de saída

Detectar:

- skill sem owner
- skill sem version
- skill sem inputs
- skill sem outputs
- skills duplicadas

---

# Runtime Validation

Estrutura recomendada:

```text
runtime/
├── profiles/
├── providers/
├── contexts/
└── execution/
```

Validar:

- segregação de ambientes
- providers válidos
- runtime isolation
- multi-provider readiness

Detectar:

- providers órfãos
- perfis inconsistentes
- execução sem contexto

---

# Telemetry Validation

Estrutura recomendada:

```text
telemetry/
├── prompts/
├── traces/
├── executions/
├── token-usage/
├── audit/
└── costs/
```

Validar:

- prompt tracing
- execution tracing
- auditoria
- token tracking
- cost tracking

Detectar:

- ausência de auditoria
- ausência de rastreabilidade
- ausência de métricas

---

# Governance Validation

Estrutura recomendada:

```text
governance/
├── architecture/
├── ai/
├── auth/
├── compliance/
├── runtime/
└── audit/
```

Validar:

- padrões corporativos
- arquitetura
- governança IA
- compliance
- auditoria

Detectar:

- governança inexistente
- padrões não documentados
- compliance ausente

---

# Policy Validation

Estrutura recomendada:

```text
policies/
├── security/
├── kubernetes/
├── sre/
├── finops/
└── compliance/
```

Validar:

- policy-as-code
- enforcement
- cobertura mínima

Detectar:

- políticas vazias
- ausência de políticas críticas

---

# Registry Validation

Estrutura recomendada:

```text
registry/
├── agents/
├── skills/
├── workflows/
├── prompts/
├── mcp/
└── auth/
```

Validar:

- consistência do catálogo
- agentes registrados
- skills registradas
- MCPs registrados

Detectar:

- ativos não registrados
- catálogo inconsistente

---

# MCP Validation

Todo MCP deve possuir:

- autenticação
- timeout
- retry
- observabilidade
- logging
- healthcheck

Validar:

- ownership
- versionamento
- autenticação
- timeout
- retry

Detectar:

- MCP sem timeout
- MCP sem retry
- MCP sem autenticação
- MCP sem observabilidade

---

# Auth Validation

Estrutura recomendada:

```text
auth/
├── auth.md
├── providers/
├── identities/
├── trust/
├── registrations/
└── lifecycle/
```

Validar:

- auth.md
- machine identity
- trust chain
- least privilege
- credential rotation
- registration workflow

Detectar:

- auth.md ausente
- trust chain ausente
- provider órfão
- token permanente

---

# AI Governance Validation

Validar:

- prompt lineage
- execution lineage
- agent lineage
- ownership
- inferência auditável
- controle de custos

Detectar:

- execução não rastreável
- prompts sem governança
- custos não monitorados

---

# Security Validation

Validar:

- secrets management
- least privilege
- RBAC
- encryption
- credential lifecycle

Detectar:

- secrets em Git
- certificados versionados
- tokens hardcoded
- credenciais fora de .secrets

---

# Kubernetes Validation

Validar:

- requests
- limits
- readinessProbe
- livenessProbe
- securityContext
- ingress
- TLS
- RBAC
- anti-affinity

Detectar:

- latest tag
- privileged=true
- root containers
- ausência de probes

---

# Workflow Validation

Validar:

- timeout
- retry
- rollback
- observabilidade
- ownership

Detectar:

- workflows órfãos
- workflows acoplados
- ausência de rollback

---

# Hooks Validation

Validar:

```text
hooks/
├── pre-command.sh
├── post-command.sh
└── validate-yaml.sh
```

Validar:

- timeout
- tratamento de erro
- idempotência

Detectar:

- hooks inseguros
- hooks sem timeout
- hooks sem tratamento de erro

---

# Configuration Validation

## opencode.jsonc

Validar:

- schema
- workspaces
- agents
- skills
- runtime
- telemetry
- mcp_servers
- hooks

Detectar:

- caminhos inválidos
- arquivos inexistentes
- configuração minimalista

## AGENTS.md

Validar:

- aderência à estrutura real
- agentes documentados
- skills documentadas
- runtime documentado

Detectar:

- documentação desatualizada

## .gitignore

Validar:

```text
.secrets/
.env
*.key
*.pem
kubeconfig*
```

---

# Anti-Patterns

Detectar:

- agentes gigantes
- skills duplicadas
- workflows acoplados
- dependências circulares
- MCP sem timeout
- MCP sem retry
- auth.md ausente
- machine identity ausente
- trust chain ausente
- governança inexistente
- policies vazias
- diretórios órfãos
- contexto global excessivo

---

# Expected Report

```text
[OK]
[WARN]
[FAIL]
[SUGGESTION]
```

---

# Scoring Model

```text
Architecture: XX/100
Governance: XX/100
Security: XX/100
Telemetry: XX/100
Runtime: XX/100
MCP: XX/100
Auth: XX/100
AI Governance: XX/100

Overall: XX/100
```

---

# Validation Principles

- Security by Default
- Governance by Design
- Observability by Default
- Policy as Code
- Infrastructure as Code
- AI Governance First
- MCP First
- Auth.md Ready
- Runtime Isolation
- Platform Engineering
- Continuous Compliance
- Enterprise Cognitive Platforms