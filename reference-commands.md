# Kubernetes Service Account Pod Management Command Reference

### Project Content Table
- [Core Operations](#core-operations)
- [Verification Commands](#verification-commands)
- [Advanced Configuration](#advanced-configuration)
- [Troubleshooting](#troubleshooting)

> **Author**: [Md Toriqul Islam](https://linkedin.com/in/thetoriqul)  
> **Description**: Complete command reference for managing Kubernetes pods with custom service accounts  
> **Note**: Review commands carefully before execution in production environments

## Core Operations

### Environment Setup
```bash
# Verify cluster access
kubectl cluster-info

# Check current context
kubectl config current-context

# Verify kubectl version
kubectl version --client
```

### Namespace Management
```bash
# Create development namespace
kubectl create ns dev1

# Verify namespace creation
kubectl get namespace dev1

# Set namespace as default for current context
kubectl config set-context --current --namespace=dev1
```

### Service Account Creation
```bash
# Create service account
kubectl create serviceaccount demo-sa -n dev1

# Verify service account creation
kubectl get serviceaccounts -n dev1

# Describe service account details
kubectl describe serviceaccount demo-sa -n dev1
```

### Pod Deployment
```bash
# Deploy pod with custom service account
kubectl run demo \
    --image=nginx \
    -n dev1 \
    --dry-run=client -o json \
    --overrides='{"spec": {"serviceAccountName": "demo-sa"}}' | kubectl apply -f -

# Verify pod deployment
kubectl get pods -n dev1

# Check pod service account configuration
kubectl get pod demo -n dev1 -o yaml | grep serviceAccount
```

## Verification Commands

### Pod Status Verification
```bash
# Check pod status
kubectl get pod demo -n dev1

# Get detailed pod information
kubectl describe pod demo -n dev1

# View pod logs
kubectl logs demo -n dev1

# Check pod events
kubectl get events -n dev1 --field-selector involvedObject.name=demo
```

### Service Account Verification
```bash
# List all service accounts
kubectl get serviceaccounts -n dev1

# Get service account details
kubectl describe serviceaccount demo-sa -n dev1

# View service account secrets
kubectl get secrets -n dev1 | grep demo-sa
```

## Advanced Configuration

### Custom Pod Configuration
```bash
# Create pod with additional parameters
kubectl run demo \
    --image=nginx \
    -n dev1 \
    --dry-run=client -o yaml \
    --overrides='{
        "spec": {
            "serviceAccountName": "demo-sa",
            "containers": [{
                "name": "nginx",
                "ports": [{"containerPort": 80}],
                "resources": {
                    "requests": {
                        "memory": "64Mi",
                        "cpu": "250m"
                    },
                    "limits": {
                        "memory": "128Mi",
                        "cpu": "500m"
                    }
                }
            }]
        }
    }' | kubectl apply -f -

# Verify configuration
kubectl get pod demo -n dev1 -o yaml
```

### Security Context Configuration
```bash
# Deploy pod with security context
kubectl run demo \
    --image=nginx \
    -n dev1 \
    --dry-run=client -o yaml \
    --overrides='{
        "spec": {
            "serviceAccountName": "demo-sa",
            "securityContext": {
                "runAsNonRoot": true,
                "runAsUser": 1000
            }
        }
    }' | kubectl apply -f -
```

## Troubleshooting

### Common Debugging Commands
```bash
# Check pod events
kubectl describe pod demo -n dev1

# View pod logs
kubectl logs demo -n dev1

# Check service account token
kubectl describe secret $(kubectl get serviceaccount demo-sa -n dev1 -o jsonpath='{.secrets[0].name}') -n dev1
```

### Cleanup Operations
```bash
# Delete pod
kubectl delete pod demo -n dev1

# Delete service account
kubectl delete serviceaccount demo-sa -n dev1

# Delete namespace and all resources
kubectl delete namespace dev1
```

## Learning Notes

1. Always verify service account existence before pod deployment
2. Use `--dry-run=client` to validate configurations
3. Check pod events for troubleshooting deployment issues
4. Implement proper RBAC policies in production
5. Regularly rotate service account tokens for security

---

> 💡 **Best Practice**: Always use dedicated service accounts for different applications/components

> ⚠️ **Warning**: Never use default service account in production environments

> 📝 **Note**: Service account tokens are automatically mounted at `/var/run/secrets/kubernetes.io/serviceaccount/`