# eks-demo-app

GitOps manifests for the Spinifex EKS demo app, synced by **Argo CD** in the
[`eks-gitops-argocd`](https://github.com/mulgadc/spinifex/tree/main/docs/terraform-workbooks/eks-gitops-argocd)
Terraform workbook.

The app itself (a small Spinifex-themed web server) is built and pushed to ECR
from the workbook's [`demo-app/`](https://github.com/mulgadc/spinifex/tree/main/docs/terraform-workbooks/demo-app)
directory. This repo holds only the Kubernetes manifests Argo CD deploys.

## Layout

```
manifests/
├── kustomization.yaml   # ties the resources together; sets the image
├── deployment.yaml      # the app, with /data mounted from the PVC
├── service.yaml         # NodePort the ALB targets
└── pvc.yaml             # gp3 EBS-CSI (Viperblock-backed) volume for state
```

The Argo CD `Application` (created by the workbook) points at `path: manifests`.

## Set your image

Edit `manifests/kustomization.yaml` and set `newName` to the ECR repository URI
the workbook prints (`tofu output ecr_repository_url`):

```yaml
images:
  - name: spinifex-demo
    newName: <account-id>.dkr.ecr.<region>.<your-spinifex-domain>/spinifex-demo
    newTag: latest
```

Commit and push — Argo CD picks up the change and re-syncs.

## Persistence

`deployment.yaml` mounts the `spinifex-demo-data` PVC at `/data` and sets
`DATA_DIR=/data`, so the app persists its hit counter to a Viperblock-backed EBS
volume. The volume is `ReadWriteOnce`, so the Deployment runs a single replica;
the count survives pod restarts.

## Render locally

```bash
kubectl kustomize manifests/
```
