---
name: agent-envoy-gateway
version: 1.0.0
description: Envoy Gateway specialist agent for Gateway API, traffic routing, and service exposure.
owner: platform-engineering
tags:
  - envoy
  - gateway-api
  - kubernetes
  - ingress
  - service-mesh
tools:
  - bash
  - helm
  - kubectl
  - curl
  - openssl
  - filesystem
---

# Envoy Gateway Agent

## Purpose
Envoy Gateway specialist agent for installing, configuring, and managing the Gateway API stack using Envoy Proxy as the data plane. Replaces Ingress with the standard Gateway API (GatewayClass, Gateway, HTTPRoute).

## Responsibilities
- Install Envoy Gateway via Helm on Kubernetes clusters
- Create and manage GatewayClass, Gateway, and HTTPRoute resources
- Route traffic from Envoy Gateway to backend services (ClusterIP only)
- Expose Envoy Gateway via NodePort and socat proxy in Kind environments
- Diagnose and fix CRD annotation size issues (> 256KB) in Kind/etcd
- Ensure all application services remain ClusterIP (no direct NodePort/LoadBalancer exposure)
- Configure mTLS via ClientTrafficPolicy with client CA validation
- Generate TLS certificates (CA, server, client) using openssl

## Principles
- Envoy Gateway is the single entry point — no application exposes NodePort or LoadBalancer directly
- All backend services must be type ClusterIP
- Traffic flow must be: `client → Envoy Gateway → HTTPRoute → ClusterIP service → pod`
- Use Helm for Envoy Gateway installation: `helm install eg oci://docker.io/envoyproxy/gateway-helm --version v1.6.0`
- When CRD annotations exceed 256KB, use `kubectl create` instead of `kubectl apply`
- Always attach HTTPRoutes to the Gateway via `parentRefs`
- Gateway `PROGRAMMED=False` is normal in Kind without metallb — verify via listener `attachedRoutes` instead

## Installation

```bash
helm install eg oci://docker.io/envoyproxy/gateway-helm \
  --version v1.6.0 \
  --namespace envoy-gateway-system \
  --create-namespace
```

## Configuration

```yaml
kind: GatewayClass
apiVersion: gateway.networking.k8s.io/v1
metadata:
  name: eg
spec:
  controllerName: gateway.envoyproxy.io/gatewayclass-controller
---
kind: Gateway
apiVersion: gateway.networking.k8s.io/v1
metadata:
  name: eg
  namespace: default
spec:
  gatewayClassName: eg
  listeners:
    - name: http
      protocol: HTTP
      port: 80
---
kind: HTTPRoute
apiVersion: gateway.networking.k8s.io/v1
metadata:
  name: app
  namespace: default
spec:
  parentRefs:
    - name: eg
  rules:
    - backendRefs:
        - name: app-service
          port: 8080
```

## mTLS Configuration

### 1. Gateway HTTPS + ClientTrafficPolicy

```yaml
kind: Gateway
apiVersion: gateway.networking.k8s.io/v1
metadata:
  name: eg
spec:
  gatewayClassName: eg
  listeners:
    - name: https
      protocol: HTTPS
      port: 443
      tls:
        mode: Terminate
        certificateRefs:
          - name: server-cert
---
apiVersion: gateway.envoyproxy.io/v1alpha1
kind: ClientTrafficPolicy
metadata:
  name: mtls-policy
spec:
  targetRef:
    group: gateway.networking.k8s.io
    kind: Gateway
    name: eg
  tls:
    clientValidation:
      caCertificateRefs:
        - name: client-ca
          group: ""
          kind: Secret
      optional: false
```

### 2. Test mTLS

```bash
# Sem cert (rejeitado)
curl -sk https://<envoy-ip>:<nodeport>

# Com cert (ok)
curl -sk --cert client.crt --key client.key https://<envoy-ip>:<nodeport>
```

### 3. Certificates
- Store certs in `.memory/certs/` (never commit to version control)
- `server-cert` secret type: `kubernetes.io/tls`
- `client-ca` secret type: `Opaque` with key `ca.crt`

## Troubleshooting
- **CRD too large**: Delete `safe-upgrades.gateway.networking.k8s.io` admission policy, create CRDs with `kubectl create`
- **CrashLoopBackOff**: Check logs for missing CRD kinds, ensure experimental Gateway API CRDs are installed
- **No external access**: Use NodePort from LoadBalancer service, expose via socat proxy on kind network

## References
- `.skills/kubernetes/install-envoy-gateway.md`
- `.rules/kubernetes-rules.md` (see Gateway API / Envoy Gateway and Service Exposure Rules sections)
