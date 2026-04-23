# Cloud-Native App for Best Buy

This repository deploys the Best Buy microservices stack to Kubernetes using GitHub Actions.

## Deploy with GitHub Actions

The workflow file is:

- `.github/workflows/deploy.yml`

It will deploy automatically when:

- You push changes to `main` in `deployment_files/**`
- You run the workflow manually from the **Actions** tab (`workflow_dispatch`)

## Prerequisites

1. A running Kubernetes cluster (AKS, EKS, GKE, or any CNCF-compliant cluster).
2. `kubectl` access from your local machine.
3. A kubeconfig file for that cluster.
4. The Kubernetes secret `best-buy-secrets` created in the target namespace (default namespace if not specified).

## Configure GitHub Secret

Create one repository secret named `KUBE_CONFIG_DATA` with your kubeconfig encoded in base64.

### PowerShell (Windows)

```powershell
$kubeconfigPath = "$HOME\.kube\config"
[Convert]::ToBase64String([IO.File]::ReadAllBytes($kubeconfigPath))
```

Copy the output and add it in GitHub:

- Repo -> Settings -> Secrets and variables -> Actions -> New repository secret
- Name: `KUBE_CONFIG_DATA`
- Value: (paste base64 output)

## Ensure Application Secrets Exist in Cluster

Your manifest expects this Kubernetes secret:

- `best-buy-secrets`

Example (replace values):

```bash
kubectl create secret generic best-buy-secrets \
	--from-literal=ASERVICE_BUS_CONNECTION_STRING="..." \
	--from-literal=MONGO_URI="..." \
	--from-literal=BLOB_CONNECTION_STRING="..."
```

## Run Deployment

1. Push to `main`, or
2. Open **Actions** -> **Deploy to Kubernetes** -> **Run workflow**.

The workflow executes:

1. Authenticate using `KUBE_CONFIG_DATA`
2. Apply [deployment_files/best-buy.yaml](deployment_files/best-buy.yaml)
3. Wait for rollout success of all deployments

## Troubleshooting

- If deployment fails at kubeconfig step, verify `KUBE_CONFIG_DATA` exists and is valid base64.
- If pods crash with missing env values, recreate/update `best-buy-secrets`.
- If rollout times out, inspect pods and events:

```bash
kubectl get pods
kubectl describe pod <pod-name>
kubectl get events --sort-by=.metadata.creationTimestamp
```
