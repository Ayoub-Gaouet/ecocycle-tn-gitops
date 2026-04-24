# ecocycle-tn-gitops

GitOps repository for EcoCycle TN Kubernetes manifests synchronized by ArgoCD.

## Structure

- `argocd/ecocycle-user-service-application.yaml`: ArgoCD `Application` created once during setup.
- `k8s/`: Kustomize base watched by ArgoCD.
- `k8s/kustomization.yaml`: image tag source updated by the backend CI job.
- `k8s/servicemonitor.yaml`: monitoring manifest kept ready for EC-9. Add it to `resources` after installing `kube-prometheus-stack` and enabling `/actuator/prometheus`.

## Local Setup

Start Minikube with the EC-8 target capacity:

```bash
minikube start --cpus=4 --memory=8192
```

Install ArgoCD once:

```bash
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```

Create the ArgoCD application once:

```bash
kubectl apply -f argocd/ecocycle-user-service-application.yaml
```

After that initial setup, do not run manual `kubectl apply` for application changes. The cluster should be reconciled from Git by ArgoCD.

## Continuous Deployment Flow

1. The backend CI builds and pushes `ghcr.io/ayoub-gaouet/ecocycle-tn:<short-sha>`.
2. The CI `update-gitops` job updates `k8s/kustomization.yaml` in this repository.
3. ArgoCD detects the Git change and syncs `ecocycle-user-service` automatically.
4. A healthy demo should show ArgoCD moving from `OutOfSync` to `Syncing` to `Synced`.

The backend repository needs a `GITOPS_PAT` GitHub secret with contents write access to this repository.

## Useful Commands

```bash
kubectl -n argocd get applications
kubectl -n ecocycle get pods,svc,ingress
kubectl -n ecocycle rollout status deployment/ecocycle-user-service
kubectl -n ecocycle port-forward svc/ecocycle-user-service 8080:80
```

Then check:

```bash
curl http://localhost:8080/actuator/health
```

For ingress on Minikube, enable the ingress addon and map `ecocycle.local` to the Minikube IP in your hosts file.
