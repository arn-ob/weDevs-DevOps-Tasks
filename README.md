# weDevs-DevOps-Tasks

Pre-Install:

```md
1. Install Ubuntu Desktop in ProxMox
2. Insall Docker
```

## Task 1: Cluster Setup and Application Deployment

### Create a Kubernetes cluster with k3s on your local machine.

#### k3s installing

```bash
curl -sfL https://get.k3s.io | sh -
```

output:

```bash
kubectl get nodes
```

```bash
NAME     STATUS   ROLES                  AGE    VERSION
ubuntu-standard-pc-i440fx-piix-1996   Ready    control-plane,master   5m36s   v1.33.4+k3s1
```

### Use FluxCD (GitOps tool) to bootstrap the Flux controller into the cluster, and configure it to sync the cluster state from a GitHub repository.

#### Installing FluxCD

```bash
curl -s https://fluxcd.io/install.sh | sudo bash
```

Install the Flux Operator

```bash
helm install flux-operator oci://ghcr.io/controlplaneio-fluxcd/charts/flux-operator \
  --namespace flux-system \
  --create-namespace
```

outptu:

```bash
root@ubuntu:/home/ubuntu# helm install flux-operator oci://ghcr.io/controlplaneio-fluxcd/charts/flux-operator   --namespace flux-system   --create-namespace
Pulled: ghcr.io/controlplaneio-fluxcd/charts/flux-operator:0.28.0
Digest: sha256:9ee42975ae78d465c7006fd98431f3e2971326f16f79940f747d997aa1984b94PJ
NAME: flux-operator
LAST DEPLOYED: Sat Sep 13 05:07:56 2025
NAMESPACE: flux-system
STATUS: deployed
REVISION: 1
TEST SUITE: None
NOTES:
Documentation at https://fluxcd.control-plane.io/operator/
```

Connecting github repository

```bash
export GITHUB_TOKEN=<token>
flux bootstrap github \
  --token-auth=false \
  --owner=arn-ob \
  --repository=arn-ob/weDevs-DevOps-Tasks \
  --branch=main \
  --path=workloads \
  --read-write-key=true \
  --personal
```

output:

```bash
► connecting to github.com
► cloning branch "main" from Git repository "https://github.com/arn-ob/weDevs-DevOps-Tasks.git"
✔ cloned repository
► generating component manifests
✔ generated component manifests
✔ committed component manifests to "main" ("8b7e6b712c00f2db1ecfdcfed1cf70f440fb76f0")
► pushing component manifests to "https://github.com/arn-ob/weDevs-DevOps-Tasks.git"
► installing components in "flux-system" namespace
✔ installed components
✔ reconciled components
► determining if source secret "flux-system/flux-system" exists
► generating source secret
✔ public key: ecdsa-sha2-nistp384 AAAAE2VjZHNhLXNoYTItbmlzdHAzODQAAAAIbmlzdHAzODQAAABhBJk0Az+nm56L8xOx2Raq76AnA/1dkvVN3QDgB1lot/JfzEZSOgFku7zw1URpJ3o+XVlUbkMnWgKV6XvarDFcam69GcVsvsGvPWBHTiKxNrX6d5jvx8988vtwgVXkCfr7/w==
✔ configured deploy key "flux-system-main-flux-system-./workloads" for "https://github.com/arn-ob/weDevs-DevOps-Tasks"
► applying source secret "flux-system/flux-system"
✔ reconciled source secret
► generating sync manifests
✔ generated sync manifests
✔ committed sync manifests to "main" ("e4ed3dd9bd2e56a86d6e69f156f74e0229f0eb4e")
► pushing sync manifests to "https://github.com/arn-ob/weDevs-DevOps-Tasks.git"
► applying sync manifests
✔ reconciled sync configuration
◎ waiting for GitRepository "flux-system/flux-system" to be reconciled
✔ GitRepository reconciled successfully
◎ waiting for Kustomization "flux-system/flux-system" to be reconciled
✔ Kustomization reconciled successfully
► confirming components are healthy
✔ helm-controller: deployment ready
✔ kustomize-controller: deployment ready
✔ notification-controller: deployment ready
✔ source-controller: deployment ready
✔ all components are healthy
```