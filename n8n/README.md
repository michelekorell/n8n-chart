# n8n Helm Chart

This Helm chart deploys n8n on a Kubernetes cluster.

## Prerequisites

- Kubernetes 1.16+
- Helm 3.0+
- PV provisioner support in the underlying infrastructure (if persistence is required)

## Installing the Chart

1. Create a values file (e.g., `my-values.yaml`) and configure the required settings:

```yaml
n8n:
  encryption:
    key: "your-32-char-encryption-key"  # Required

  # Optional: Configure basic authentication
  basicAuth:
    enabled: true
    user: "admin"
    password: "your-password"

  # Optional: Configure external database (default: sqlite)
  database:
    type: "postgresdb"  # Options: sqlite, postgresdb, mysqldb
    postgresdb:
      host: "your-postgres-host"
      port: 5432
      database: "n8n"
      user: "n8n"
      password: "your-db-password"

  # Optional: Configure SMTP
  smtp:
    enabled: true
    host: "smtp.example.com"
    port: 587
    user: "your-smtp-user"
    password: "your-smtp-password"
    sender: "n8n@example.com"
    ssl: false

# Configure persistence
persistence:
  enabled: true
  storageClass: "standard"  # Use your storage class
  size: 10Gi

# Configure ingress
ingress:
  enabled: true
  className: "nginx"
  annotations:
    kubernetes.io/ingress.class: nginx
  hosts:
    - host: n8n.example.com
      paths:
        - path: /
          pathType: Prefix
```

2. Install the chart:

```bash
helm install my-n8n ./n8n -f my-values.yaml
```

## Configuration

The following table lists the configurable parameters of the n8n chart and their default values.

| Parameter | Description | Default |
|-----------|-------------|---------|
| `replicaCount` | Number of n8n replicas | `1` |
| `image.repository` | n8n image repository | `n8nio/n8n` |
| `image.tag` | n8n image tag | `""` (defaults to chart appVersion) |
| `image.pullPolicy` | Image pull policy | `IfNotPresent` |
| `n8n.encryption.key` | Encryption key for n8n | `""` |
| `n8n.basicAuth.enabled` | Enable basic authentication | `false` |
| `n8n.basicAuth.user` | Basic auth username | `""` |
| `n8n.basicAuth.password` | Basic auth password | `""` |
| `n8n.database.type` | Database type (sqlite, postgresdb, mysqldb) | `sqlite` |
| `persistence.enabled` | Enable persistence | `true` |
| `persistence.storageClass` | Storage class for PVC | `""` |
| `persistence.size` | Size of PVC | `10Gi` |
| `ingress.enabled` | Enable ingress | `false` |
| `resources` | CPU/Memory resource requests/limits | `{}` |

## Upgrading

To upgrade the release:

```bash
helm upgrade my-n8n ./n8n -f my-values.yaml
```

## Uninstalling

To uninstall/delete the deployment:

```bash
helm uninstall my-n8n
```

## Notes

- The default configuration uses SQLite as the database. For production environments, it's recommended to use PostgreSQL or MySQL.
- Make sure to set a secure encryption key before deploying.
- If you enable persistence, make sure your cluster has the required storage class available.
- For production use, it's recommended to enable ingress with proper TLS configuration. 