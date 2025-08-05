# n8n Helm Chart

A Helm chart for deploying n8n workflow automation tool on Kubernetes with support for multiple versions and Rancher integration.

## Features

- **Multiple Version Support**: Deploy different n8n versions (stable and latest)
- **Rancher Integration**: Optimized for Rancher catalog with user-friendly UI
- **Production Ready**: Includes security, persistence, and monitoring configurations
- **Flexible Database Support**: SQLite (default), PostgreSQL, and MySQL
- **Customizable Resources**: Configurable CPU and memory limits
- **Ingress Support**: Built-in ingress configuration for external access

## Available Versions

| Version | Stability | Description |
|---------|-----------|-------------|
| `1.105.3` | ✅ Stable | **Default** - Latest stable version (recommended for production) |
| `1.85.4` | ✅ Stable | Previous stable version |
| `1.84.3` | ✅ Stable | Previous stable version |
| `1.83.2` | ✅ Stable | Previous stable version |
| `1.82.3` | ✅ Stable | Previous stable version |
| `latest` | ⚠️ Development | Latest development version (use with caution) |

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

# Install with specific version
helm install n8n n8n-chart/n8n \
  --set image.tag="1.84.3" \
  --set n8n.encryption.key="your-encryption-key"

# Install with latest version (development)
helm install n8n n8n-chart/n8n \
  --set image.tag="latest" \
  --set n8n.encryption.key="your-encryption-key"
```

### Using Rancher Catalog

1. Add this chart to your Rancher catalog
2. Navigate to Apps & Marketplace in Rancher
3. Find "n8n Workflow Automation" and click Install
4. Configure the following required fields:
   - **n8n Version**: Select your preferred version (1.105.3 recommended)
   - **Encryption Key**: Required for data security
   - **Basic Auth**: Enable and set credentials for security

## Configuration

### Version Selection

The chart supports multiple n8n versions through the `image.tag` parameter:

```yaml
image:
  repository: docker.n8n.io/n8nio/n8n
  tag: "1.105.3"  # Default stable version
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
    type: sqlite  # Options: sqlite, postgresdb, mysqldb
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

### MySQL

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

### Between Versions

```bash
# Upgrade to a newer version
helm upgrade n8n n8n-chart/n8n --set image.tag="1.105.3"

# Rollback if needed
helm rollback n8n
```

### Version Compatibility

- Always backup your data before upgrading
- Test upgrades in a non-production environment first
- Check n8n release notes for breaking changes

## Support

For issues and questions:

- Chart issues: Create an issue in this repository
- n8n issues: Check [n8n documentation](https://docs.n8n.io/)
- Rancher issues: Check [Rancher documentation](https://rancher.com/docs/)

## License

This chart is licensed under the Apache 2.0 License. 