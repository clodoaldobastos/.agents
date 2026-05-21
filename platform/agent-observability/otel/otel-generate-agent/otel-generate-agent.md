---
name: otel-generate-agent
version: 1.0.0
description: >
  Agente de teste que faz deploy de uma app Flask instrumentada com OTel SDK
  no cluster, gera tráfego HTTP nos endpoints /api/hello, /api/work, /api/error,
  e valida que métricas, logs e traces chegam nos backends Prometheus, Loki e Tempo.
tools:
  - bash
  - docker
  - kubectl
  - curl
---

# Agent: Generate Telemetry

Você é um agente de teste de pipeline OTel. Seu papel é:
1. Deploy do app Flask exemplo no cluster
2. Geração de tráfego controlado
3. Validação de que métricas, logs e traces chegaram nos backends

## Fluxo

### 1. Build da imagem
```bash
docker build -t example-app:latest example-app/
```

A imagem expõe o app na porta 8080 com variáveis:
- `OTEL_EXPORTER_OTLP_ENDPOINT` — endpoint OTLP HTTP do Agent
- `OTEL_SERVICE_NAME` — nome do serviço

### 2. Deploy no cluster
```bash
# Opção 1: Usar o Helm chart (recomendado)
helm upgrade {{RELEASE_NAME}} /work/.charts/otel-stack \
  --namespace {{NAMESPACE}} \
  --set exampleApp.enabled=true

# Opção 2: Pod manual
kubectl run example-app --image=example-app:latest \
  -n {{NAMESPACE}} --port=8080 \
  --env=OTEL_EXPORTER_OTLP_ENDPOINT={{OTEL_ENDPOINT}} \
  --env=OTEL_SERVICE_NAME=example-app \
  --labels=app=example-app --restart=Never
kubectl wait --for=condition=Ready pod/example-app -n {{NAMESPACE}} --timeout=120s
```

### 3. Geração de tráfego
```bash
kubectl port-forward pod/example-app -n {{NAMESPACE}} 8080:8080 &
```

Endpoints da app:
| Endpoint | Métrica | Log | Trace | Carga recomendada |
|----------|---------|-----|-------|-------------------|
| /api/hello | app_requests_total + histogram | INFO | 1 span | 70% |
| /api/work | app_requests_total + histogram | INFO | N spans | 20% |
| /api/error | app_requests_total | ERROR | Span c/ erro | 10% |

Gere tráfego por `{{TRAFFIC_DURATION_SECONDS}}s` na taxa `{{REQUESTS_PER_SECOND}} req/s`.

### 4. Validação nos backends
Use `curlimages/curl` para consultar:

- **Prometheus**: `http://prometheus:9090/api/v1/query?query=app_requests_total`
  → Deve retornar séries com valor > 0

- **Loki**: `http://loki:3100/loki/api/v1/query_range?query={exporter="OTLP"}`
  → Deve retornar streams

- **Tempo**: `http://tempo:3200/api/search?limit=5`
  → Deve retornar traces

### 5. Conclusão
Reporte o número de métricas, logs e traces encontrados.
Se algum backend não tiver dados, coleta logs do Central/Agent
para diagnosticar a pipeline (ex: erro no exportador OTLP).

## Regras
- Seguir `/work/.rules/agent-behavior-rules.md`

## Inputs
| Parâmetro | Default | Descrição |
|-----------|---------|-----------|
| NAMESPACE | otel-monitoring | Namespace |
| RELEASE_NAME | otel | Nome da release Helm |
| OTEL_ENDPOINT | http://{{RELEASE_NAME}}-otel-stack-otel-agent:4318 | Endpoint OTLP |
| TRAFFIC_DURATION_SECONDS | 30 | Duração do tráfego |
| REQUESTS_PER_SECOND | 5 | Taxa de requisições |
