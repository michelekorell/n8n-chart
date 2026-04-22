# n8n Helm Chart

A Helm chart for deploying n8n workflow automation tool on Kubernetes with support for multiple versions and Rancher integration.

## Features

- **Multiple Version Support**: Deploy different n8n versions (stable and latest)
- **Rancher Integration**: Optimized for Rancher catalog with user-friendly UI
- **Production Ready**: Includes security, persistence, and monitoring configurations
- **Flexible Database Support**: SQLite (default) and PostgreSQL; MySQL as application database is supported only for **n8n 1.x** images (n8n 2.0+ [dropped MySQL](https://docs.n8n.io/2-0-breaking-changes/))
- **Customizable Resources**: Configurable CPU and memory limits
- **Ingress Support**: Built-in ingress configuration for external access

## Chart version vs n8n version

The Helm chart `version` (for example `3.0.0`) is independent of the n8n Docker image tag. `Chart.yaml` `appVersion` and `catalog.cattle.io/upstream-version` describe the **default** n8n version in `values.yaml`; you can override the image with `n8nVersion` / `image.tag`.

## Available Versions (image tags)

| Version | Role | Description |
|---------|------|-------------|
| `2.17.3` | **Default stable** | n8n **2.x** — default when `n8nVersion.selection` is `stable` |
| `1.123.16` | 1.x | Latest **n8n 1.x** (use with `selection: specific` to stay on 1.x) |
| `1.121.3` | 1.x | Earlier 1.x |
| `1.106.3` … `1.82.3` | 1.x | Older 1.x for rollback |
| `latest` | ⚠️ Development | Floating tag (use with caution) |

## Quick Start

### Using Helm CLI

```bash
# Add the repository
helm repo add n8n-chart https://your-repo-url
helm repo update

# Install with stable version (default)
helm install n8n n8n-chart/n8n \
  --set n8n.encryption.key="your-encryption-key" \
  --set n8n.basicAuth.enabled=true \
  --set n8n.basicAuth.user="admin" \
  --set n8n.basicAuth.password="your-password"

# Install with a specific version (must set selection to specific)
helm install n8n n8n-chart/n8n \
  --set n8nVersion.selection=specific \
  --set image.tag="1.123.16" \
  --set n8n.encryption.key="your-encryption-key"

# Install with latest version (development)
helm install n8n n8n-chart/n8n \
  --set n8nVersion.selection=specific \
  --set image.tag="latest" \
  --set n8n.encryption.key="your-encryption-key"
```

### Using Rancher Catalog

1. Add this chart to your Rancher catalog
2. Navigate to Apps & Marketplace in Rancher
3. Find "n8n Workflow Automation" and click Install
4. Configure the following required fields:
   - **Version Selection**: Choose "stable" (recommended) or "specific"
   - **n8n Version**: Select specific version (only shown if "specific" is chosen)
   - **Encryption Key**: Required for data security
   - **Basic Auth**: Enable and set credentials for security

## Configuration

### Version Selection

The chart supports two version selection modes:

1. **Stable** (default): Uses `n8nVersion.default` (n8n 2.x unless you change it)
2. **Specific**: Uses `image.tag` (for example `1.123.16` to stay on latest 1.x)

```yaml
n8nVersion:
  selection: "stable"  # Options: "stable" or "specific"
  default: "2.17.3"   # Used when selection is "stable"

image:
  repository: docker.n8n.io/n8nio/n8n
  tag: "2.17.3"  # Used when selection is "specific"
```

### Essential Configuration

```yaml
# Required: Encryption key for n8n data
n8n:
  encryption:
    key: "your-secure-encryption-key"

# Recommended: Basic authentication
  basicAuth:
    enabled: true
    user: "admin"
    password: "your-secure-password"

# Database configuration (SQLite is default)
  database:
    type: sqlite  # Options: sqlite, postgresdb, mysqldb (mysqldb only with n8n 1.x images)
  # Optional (n8n 2.x+): set to "true" or "false" to override N8N_ENFORCE_SETTINGS_FILE_PERMISSIONS (e.g. if the pod fails on volume permissions)
  # enforceSettingsFilePermissions: false  # optional boolean or string; omit for default
```

### Resource Configuration

```yaml
resources:
  limits:
    cpu: 1000m
    memory: 4096Mi
  requests:
    cpu: 500m
    memory: 512Mi
```

### Persistence

```yaml
persistence:
  enabled: true
  size: 20Gi
  storageClass: ""  # Use default storage class
```

### Ingress Configuration

```yaml
ingress:
  enabled: true
  domainName: "n8n.yourdomain.com"
```

## Version-Specific Values Files

### Stable Version (Recommended for Production)

```bash
helm install n8n n8n-chart/n8n -f values-stable.yaml
```

### Latest Version (Development/Testing)

```bash
helm install n8n n8n-chart/n8n -f values-latest.yaml
```

## Database Configuration

### SQLite (Default)

No additional configuration required. Data is stored in persistent volume.

### PostgreSQL

```yaml
n8n:
  database:
    type: postgresdb
    postgresdb:
      host: "your-postgres-host"
      port: 5432
      database: n8n
      user: n8n
      password: "your-postgres-password"
```

### MySQL (n8n 1.x only)

n8n **2.0+** no longer supports MySQL as the application database. This chart will **refuse to render** a deployment that combines `mysqldb` with a 2.x image tag. For n8n 1.x, you can still use:

```yaml
n8n:
  database:
    type: mysqldb
    mysqldb:
      host: "your-mysql-host"
      port: 3306
      database: n8n
      user: n8n
      password: "your-mysql-password"
```

Use a **1.x** `image.tag` (for example `1.123.16`) with `n8nVersion.selection: specific`.

## Security Considerations

1. **Encryption Key**: Always set a secure encryption key
2. **Basic Auth**: Enable basic authentication for production deployments
3. **Network Security**: Use ingress with TLS for external access
4. **Resource Limits**: Set appropriate resource limits to prevent resource exhaustion

## Troubleshooting

### Common Issues

1. **Pod fails to start**: Check encryption key is set
2. **Database connection issues**: Verify database credentials and connectivity
3. **Resource constraints**: Increase CPU/memory limits if needed
4. **Ingress not working**: Verify ingress controller is installed and configured

### Logs

```bash
# Check pod logs
kubectl logs -f deployment/n8n

# Check pod status
kubectl get pods -l app.kubernetes.io/name=n8n
```

## Upgrading

### Between versions

Back up the n8n data (PVC and/or workflow exports) before major upgrades, especially when moving from n8n 1.x to 2.x.

```bash
helm upgrade n8n n8n-chart/n8n

# Rollback if needed
helm rollback n8n
```

### n8n 1.x to 2.x

- Read [n8n v2.0 breaking changes](https://docs.n8n.io/2-0-breaking-changes/) (workflows, publishing, MySQL, security defaults, and more).
- If you use **MySQL** as the n8n application database, migrate to **PostgreSQL** (or SQLite) before using a 2.x image.
- **Rancher**: push the new chart to your cluster catalog, open the **n8n** app, **Upgrade**, select chart `3.x`, and confirm values (encryption key, ingress, `n8n.database.type`).

### Version compatibility

- Always backup your data before upgrading
- Test upgrades in a non-production environment first
- Check [n8n release notes](https://docs.n8n.io/release-notes/) for breaking changes

## Support

For issues and questions:

- Chart issues: Create an issue in this repository
- n8n issues: Check [n8n documentation](https://docs.n8n.io/)
- Rancher issues: Check [Rancher documentation](https://rancher.com/docs/)

## License

This chart is licensed under the Apache 2.0 License. 