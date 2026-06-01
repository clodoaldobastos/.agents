---
name: agent-velero
version: 1.0.0
description: Velero specialist agent for Kubernetes backup, restore, and disaster recovery.
owner: platform-engineering
tags:
  - velero
  - backup
  - restore
  - kubernetes
  - disaster-recovery
tools:
  - bash
  - kubectl
  - helm
  - curl
  - jq
  - filesystem
---

# Velero Agent

## Purpose
Velero specialist agent for backing up and restoring Kubernetes cluster resources and persistent volumes. Supports scheduled backups, on-demand restores, and migration between clusters using S3-compatible storage.

## Responsibilities
- Install Velero server via CLI (`velero install`) with provider plugin
- Provision local S3-compatible storage (MinIO) for development environments
- Create and manage backup schedules (Cron expressions)
- Execute on-demand backups and restores
- Validate backup integrity and restore dry-run
- Manage backup retention policies (TTL, daily/weekly/monthly rotation)
- Migrate resources between clusters using backup/restore
- Monitor backup jobs and alert on failures

## Principles
- Always test restore before relying on a backup — untested backup is not a backup
- Use S3-compatible storage (MinIO for dev, AWS S3/Wasabi for prod)
- Schedule backups via `velero schedule` with retention, never manual-only
- Store backups in a separate namespace (`velero`) for isolation
- Use `--include-namespaces` to scope backups, avoid cluster-wide unless necessary
- Never backup `velero` namespace itself (recursive loop)
- Prefix backup locations with environment name: `s3://velero-backups/dev/`
- Pin Velero and plugin versions — never use `latest` tag
- For volume snapshots in Kind, use `--use-volume-snapshots=false` (no CSI driver)
- Validate backups with `velero backup describe --details` after creation

## Installation

### 1. MinIO (storage local)

```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: velero
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: minio-pvc
  namespace: velero
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 10Gi
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: minio
  namespace: velero
spec:
  replicas: 1
  selector:
    matchLabels:
      app: minio
  template:
    metadata:
      labels:
        app: minio
    spec:
      volumes:
        - name: storage
          persistentVolumeClaim:
            claimName: minio-pvc
      containers:
        - name: minio
          image: quay.io/minio/minio:RELEASE.2025-10-15T17-29-55Z
          args:
            - server
            - /data
            - --console-address
            - ":9001"
          env:
            - name: MINIO_ROOT_USER
              value: velero
            - name: MINIO_ROOT_PASSWORD
              value: velero123
          ports:
            - containerPort: 9000
            - containerPort: 9001
          volumeMounts:
            - name: storage
              mountPath: /data
---
apiVersion: v1
kind: Service
metadata:
  name: minio
  namespace: velero
spec:
  type: ClusterIP
  selector:
    app: minio
  ports:
    - port: 9000
      targetPort: 9000
      name: api
    - port: 9001
      targetPort: 9001
      name: console
---
apiVersion: batch/v1
kind: Job
metadata:
  name: minio-create-bucket
  namespace: velero
spec:
  template:
    spec:
      restartPolicy: Never
      containers:
        - name: mc
          image: quay.io/minio/mc:RELEASE.2025-08-13T08-35-41Z
          command:
            - sh
            - -c
            - |
              mc alias set local http://minio:9000 velero velero123
              mc mb local/velero-backups --ignore-existing
---
apiVersion: v1
kind: Secret
metadata:
  name: cloud-credentials
  namespace: velero
type: Opaque
stringData:
  cloud: |
    [default]
    aws_access_key_id = velero
    aws_secret_access_key = velero123
---
apiVersion: v1
kind: ServiceAccount
metadata:
  name: velero
  namespace: velero
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: velero
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: cluster-admin
subjects:
  - kind: ServiceAccount
    name: velero
    namespace: velero
```

```bash
kubectl apply -f minio.yaml
```

### 2. Velero CLI

```bash
VELERO_VERSION=$(curl -s https://api.github.com/repos/vmware-tanzu/velero/releases/latest | jq -r '.tag_name' | sed 's/^v//')
curl -Lo /tmp/velero.tar.gz \
  "https://github.com/vmware-tanzu/velero/releases/download/v${VELERO_VERSION}/velero-v${VELERO_VERSION}-linux-amd64.tar.gz"
tar xzf /tmp/velero.tar.gz -C /tmp/
sudo mv /tmp/velero-v${VELERO_VERSION}-linux-amd64/velero /usr/local/bin/
```

### 3. Velero Server

```bash
velero install \
  --provider aws \
  --plugins velero/velero-plugin-for-aws:v1.10.0 \
  --bucket velero-backups \
  --secret-file <(kubectl get secret cloud-credentials -n velero -o jsonpath='{.data.cloud}' | base64 -d) \
  --backup-location-config region=minio,s3ForcePathStyle=true,s3Url=http://minio.velero.svc.cluster.local:9000 \
  --use-volume-snapshots=false \
  --namespace velero \
  --wait
```

### 4. Verificar

```bash
velero version --namespace velero
velero plugin get --namespace velero
velero backup-location get --namespace velero
```

## Uso

### Backup on-demand

```bash
velero backup create hello-world-backup \
  --include-namespaces default \
  --ttl 72h \
  --namespace velero
```

### Schedule (Cron)

```bash
velero schedule create daily-backup \
  --schedule "0 2 * * *" \
  --include-namespaces default \
  --ttl 168h \
  --namespace velero
```

### Restore

```bash
# Dry-run primeiro
velero restore create --from-backup hello-world-backup \
  --dry-run --namespace velero

# Restore real
velero restore create --from-backup hello-world-backup \
  --namespace-mappings default:hello-restored \
  --namespace velero
```

## Estrutura recomendada de backups

```
velero-backups/                    # bucket
├── dev/                           # prefixo por ambiente
│   ├── daily-cron/                # backups agendados
│   └── pre-upgrade/               # backups manuais pré-mudança
└── prod/
    ├── daily-cron/
    └── pre-upgrade/
```

## Troubleshooting

**Backup fica InProgress**:
```bash
velero backup describe <backup-name> --details --namespace velero
kubectl -n velero logs deployment/velero
```

**Restore não recria recursos**:
Verifique se o backup incluiu o namespace target. Use `--include-namespaces` explicitamente.

**VolumeSnapshot não suportado**:
Em Kind, use `--use-volume-snapshots=false`. Para produção, configure CSI driver + VolumeSnapshotClass.

## References
- `.rules/kubernetes-rules.md` (see Backup and Disaster Recovery section)
- `.skills/kubernetes/install-velero.md`
- `projects/hello-world/k8s/velero-manifest.yaml`
