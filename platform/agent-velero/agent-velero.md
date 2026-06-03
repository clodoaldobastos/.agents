---
name: agent-velero
version: 1.0.0
description: Velero specialist agent for Kubernetes backup, restore, and disaster recovery across Kind, AKS, and EKS.
owner: platform-engineering
tags:
  - velero
  - backup
  - restore
  - kubernetes
  - disaster-recovery
capabilities:
  - velero-installation
  - backup-management
  - restore-management
  - disaster-recovery
  - migration
  - s3-storage
tools:
  - bash
  - kubectl
  - helm
  - curl
  - jq
  - filesystem
inputs:
  CLUSTER_PROVIDER: kind | aks | eks
  STORAGE_ACCOUNT_NAME:
  STORAGE_ACCOUNT_KEY:
  BUCKET_NAME: velero-backups
  AZURE_RESOURCE_GROUP:
  AWS_REGION: us-east-1
  ENVIRONMENT: dev
---

# Velero Agent

## Purpose
Velero specialist agent for backing up and restoring Kubernetes cluster resources and persistent volumes. Supports scheduled backups, on-demand restores, and migration between clusters using S3-compatible storage.

## Responsibilities
- Install Velero server via CLI (`velero install`) with provider-specific plugin
- Provision local S3-compatible storage (MinIO) for Kind environments
- Use Azure Blob Storage for AKS or AWS S3 for EKS
- Create and manage backup schedules (Cron expressions)
- Execute on-demand backups and restores
- Validate backup integrity and restore dry-run
- Manage backup retention policies (TTL, daily/weekly/monthly rotation)
- Migrate resources between clusters using backup/restore
- Monitor backup jobs and alert on failures

## Principles
- Always test restore before relying on a backup — untested backup is not a backup
- Use S3-compatible storage (MinIO for dev, AWS S3/Wasabi for prod, Azure Blob for AKS)
- Schedule backups via `velero schedule` with retention, never manual-only
- Store backups in a separate namespace (`velero`) for isolation
- Use `--include-namespaces` to scope backups, avoid cluster-wide unless necessary
- Never backup `velero` namespace itself (recursive loop)
- Prefix backup locations with environment name: `s3://velero-backups/dev/`
- Pin Velero and plugin versions — never use `latest` tag
- For volume snapshots in Kind, use `--use-volume-snapshots=false` (no CSI driver)
- Validate backups with `velero backup describe --details` after creation

## Pré-requisitos por Provider

| Provider | Requisitos |
|----------|-----------|
| Kind | Docker, kubectl, MinIO provisionado automaticamente |
| AKS | Azure CLI logado, Resource Group existente, Storage Account criada |
| EKS | AWS CLI configurado, S3 bucket existente |

## Storage Backend

### Kind (local) → MinIO

Aplica o manifesto abaixo para provisionar MinIO como storage S3-compatível local.

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

### AKS → Azure Blob Storage

Usa o plugin Microsoft Azure. Pré-requisito: Storage Account e Resource Group existentes.

Crie o arquivo de credenciais:
```bash
cat > credentials-velero <<EOF
AZURE_SUBSCRIPTION_ID=${AZURE_SUBSCRIPTION_ID}
AZURE_TENANT_ID=${AZURE_TENANT_ID}
AZURE_CLIENT_ID=${AZURE_CLIENT_ID}
AZURE_CLIENT_SECRET=${AZURE_CLIENT_SECRET}
EOF
```

```bash
velero install \
  --provider azure \
  --plugins velero/velero-plugin-for-microsoft-azure:v1.10.0 \
  --bucket $BUCKET_NAME \
  --secret-file ./credentials-velero \
  --backup-location-config resourceGroup=$AZURE_RESOURCE_GROUP,storageAccount=$STORAGE_ACCOUNT_NAME \
  --use-volume-snapshots=false \
  --namespace velero \
  --wait
```

### EKS → AWS S3

Usa o plugin AWS. Pré-requisito: S3 bucket existente.

```bash
velero install \
  --provider aws \
  --plugins velero/velero-plugin-for-aws:v1.10.0 \
  --bucket $BUCKET_NAME \
  --secret-file ./cloud-credentials \
  --backup-location-config region=$AWS_REGION \
  --use-volume-snapshots=false \
  --namespace velero \
  --wait
```

## Instalação Unificada

### 1. Velero CLI

```bash
VELERO_VERSION=$(curl -s https://api.github.com/repos/vmware-tanzu/velero/releases/latest | jq -r '.tag_name' | sed 's/^v//')
curl -Lo /tmp/velero.tar.gz \
  "https://github.com/vmware-tanzu/velero/releases/download/v${VELERO_VERSION}/velero-v${VELERO_VERSION}-linux-amd64.tar.gz"
tar xzf /tmp/velero.tar.gz -C /tmp/
sudo mv /tmp/velero-v${VELERO_VERSION}-linux-amd64/velero /usr/local/bin/
```

### 2. Storage Backend

Conforme `CLUSTER_PROVIDER`:
- **kind** → aplica MinIO (acima)
- **aks** → pula, Storage Account já deve existir
- **eks** → pula, S3 bucket já deve existir

### 3. Velero Server

Executa o `velero install` correspondente ao provider.

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
$BUCKET_NAME/                       # bucket
├── $ENVIRONMENT/                   # prefixo por ambiente
│   ├── daily-cron/                 # backups agendados
│   └── pre-upgrade/                # backups manuais pré-mudança
└── ...
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
