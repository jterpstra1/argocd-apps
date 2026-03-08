# Argo CD application definitions

The "app of apps" pattern is a method used in Argo CD to manage multiple applications as a single application.  
This pattern involves creating a root application that references other applications allowing you to manage and deploy
them collectively.

This repo uses Kustomize overlays to support multiple clusters from the same base configurations.

For more details on the "app of apps" pattern, refer to
the [official documentation](https://argo-cd.readthedocs.io/en/latest/operator-manual/cluster-bootstrapping/).

To ensure separation of concerns three repositories are used to manage the Argo CD setup:

* Argo CD installation and
  configuration: [https://github.com/jterpstra1/argocd-setup](https://github.com/jterpstra1/argocd-setup)
* Argo CD application definitions: [https://github.com/jterpstra1/argocd-apps](https://github.com/jterpstra1/argocd-apps)
* Kubernetes manifests or Helm charts used by Argo CD
  applications: [https://github.com/jterpstra1/argocd-k8s-resources](https://github.com/jterpstra1/argocd-k8s-resources)

## Structure

```
argocd-apps/
├── root-app/
│   ├── base/                    # Root app template
│   └── overlays/
│       ├── prod/                # Production cluster root app
│       └── vdgt-k3s/            # vdgt-k3s cluster root app
├── manifests/
│   ├── base/                    # Shared Application template
│   ├── overlays/
│   │   ├── prod/                # Apps deployed to prod cluster
│   │   └── vdgt-k3s/            # Apps deployed to vdgt-k3s cluster
│   ├── metallb/overlays/prod/
│   ├── ingress-nginx/overlays/prod/
│   └── ...                      # Per-app overlays
```

## Usage

### Bootstrap a new cluster

Generate and apply the root-app for your cluster:

```bash
# For prod cluster
kustomize build argocd-apps/root-app/overlays/prod | kubectl apply -n argocd -f -

# For vdgt-k3s cluster
kustomize build argocd-apps/root-app/overlays/vdgt-k3s | kubectl apply -n argocd -f -
```

### Add apps to a cluster

Edit the cluster's overlay kustomization to include apps:

```yaml
# argocd-apps/manifests/overlays/vdgt-k3s/kustomization.yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization

resources:
- ../../metallb/overlays/vdgt-k3s
- ../../ingress-nginx/overlays/vdgt-k3s
```

### Add a new cluster

1. Create `argocd-apps/manifests/overlays/<cluster>/kustomization.yaml` with empty resources
2. Create `argocd-apps/root-app/overlays/<cluster>/kustomization.yaml` pointing to the manifests overlay
3. Create per-app overlays in `<app>/overlays/<cluster>/` as needed