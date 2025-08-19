# Argocd GitOps Repository

This repository contains the GitOps configuration for applications using Argo CD ApplicationSets with a multi-source pattern.

## Repository Structure

```
.
├── charts/                           # Reusable Helm charts
│   ├── frontend/                     # Frontend React/Angular application chart
│   ├── backend-api/                  # Backend API service chart
│   └── database/                     # PostgreSQL database chart
│
└── applications/                     # Environment-specific values
    ├── staging/                      # Staging environment configurations
    │   ├── frontend/values.yaml
    │   ├── backend-api/values.yaml
    │   └── database/values.yaml
    └── prod/                         # Production environment configurations
        ├── frontend/values.yaml
        ├── backend-api/values.yaml
        └── database/values.yaml
```

## How It Works

1. **Argo CD ApplicationSet** scans the `applications/{environment}/` directories
2. For each application directory found (e.g., `frontend`), it creates an Argo CD Application
3. Each Application uses **two sources**:
   - **Chart source**: The Helm chart from `charts/{app-name}/`
   - **Values source**: Environment-specific values from `applications/{environment}/{app-name}/values.yaml`
4. Argo CD combines the chart with the environment values to deploy the application

## Sample Applications

### Frontend
- **Technology**: Nginx serving static files
- **Features**: Ingress configuration, horizontal pod autoscaling (prod only)
- **Environments**: Different replica counts and resource limits per environment

### Backend API
- **Technology**: Node.js application
- **Features**: Database connectivity, Redis integration, JWT authentication
- **Environments**: Different scaling parameters and environment variables

### Database
- **Technology**: PostgreSQL 15
- **Features**: Persistent storage, health checks, environment-specific configurations
- **Environments**: Different storage sizes and resource allocations

## Adding a New Application

1. **Create the Helm chart**:
   ```bash
   mkdir -p charts/my-new-app/templates
   # Add Chart.yaml, values.yaml, and template files
   ```

2. **Add environment-specific values**:
   ```bash
   mkdir -p applications/staging/my-new-app
   mkdir -p applications/prod/my-new-app
   # Add values.yaml files for each environment
   ```

3. **Commit and push**:
   ```bash
   git add charts/my-new-app/ applications/*/my-new-app/
   git commit -m "feat: Add my-new-app"
   git push
   ```

4. **Automatic deployment**: Argo CD ApplicationSet will detect the new application and deploy it within minutes.

## Environment Differences

### Staging
- Lower resource allocations
- Smaller persistent volumes
- Debug logging enabled
- Single replica (except for backend-api)

### Production
- Higher resource allocations
- Larger persistent volumes
- Production logging levels
- Multiple replicas with anti-affinity rules
- Horizontal pod autoscaling enabled

## Security Considerations

⚠️ **Important**: The values files in this repository contain placeholder passwords and secrets. In a real deployment:

1. Use **Kubernetes Secrets** or **External Secrets Operator**
2. Integrate with **Google Secret Manager**, **AWS Secrets Manager**, or **HashiCorp Vault**
3. Never commit real secrets to Git repositories
4. Use **sealed-secrets** or similar tools for GitOps-compatible secret management

## Local Development

To test charts locally:

```bash
# Validate chart syntax
helm lint charts/frontend/

# Render templates with staging values
helm template frontend charts/frontend/ -f applications/staging/frontend/values.yaml

# Install to local cluster
helm install frontend-staging charts/frontend/ -f applications/staging/frontend/values.yaml --namespace staging --create-namespace
```

## Monitoring and Troubleshooting

```bash
# Check ApplicationSet status
kubectl get applicationsets -n argocd

# View generated applications
kubectl get applications -n argocd

# Check application sync status
kubectl describe application frontend-staging -n argocd
```

This structure provides a clean separation between application logic (charts) and environment-specific configuration (values), making it easy to manage multiple environments with the same codebase.
