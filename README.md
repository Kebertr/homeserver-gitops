# Homeserver GitOps

This repository contains the declarative Kubernetes configuration for a personal K3s cluster.

Argo CD watches this repository and reconciles the cluster with the desired state defined by its Applications and Helm charts.

## Architecture

```text
Git repository
      |
      v
   Argo CD
      |
      v
Helm charts and Kubernetes resources
      |
      v
    K3s cluster
```

Public traffic enters through Cloudflare Tunnel and is forwarded to either Kong or Traefik. Private administrative services are exposed through Tailscale.

## Repository structure

```text
applications/        Argo CD Application definitions
helm_charts/         Helm charts for cluster workloads
argocd-install/      Argo CD installation chart
```

Valhall has separate development and production Applications under:

```text
applications/valhall/
```

## Managed applications

The repository currently includes configuration for:

- Argo CD
- Argo CD Image Updater
- Cloudflare Tunnel
- Kong Gateway
- Tailscale
- MinIO
- Keycloak
- Prometheus
- Grafana
- Node Exporter
- GitHub Actions runners
- Valhall frontend
- Valhall member API
- Valhall bong API
- Valhall PostgreSQL databases

## Traffic flow

A public request can follow this route:

```text
Internet
   |
Cloudflare
   |
Cloudflare Tunnel
   |
Kong or Traefik
   |
Kubernetes Service
   |
Application Pod
```

The hostname configured in Cloudflare Tunnel decides which ingress controller receives the request.

The Kubernetes `ingressClassName` then decides which controller owns an Ingress:

```yaml
spec:
  ingressClassName: kong
```

For example, the public MinIO upload endpoint uses Kong, while the MinIO console remains private through Tailscale.

## Prerequisites

Install and configure:

- A running Kubernetes or K3s cluster
- `kubectl`
- Helm
- Argo CD
- Access to the cluster through `~/.kube/config`
- Required Kubernetes Secrets

## Bootstrap Argo CD

Create the namespace:

```bash
kubectl create namespace argocd
```

Install Argo CD:

```bash
kubectl apply -n argocd --server-side --force-conflicts \
  -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```

Apply an Argo CD Application:

```bash
kubectl apply -f applications/<application>.yaml
```

For example:

```bash
kubectl apply -f applications/minio-app.yaml
```

Argo CD will then render the configured Helm chart and reconcile its resources.

## Helm charts

Charts are stored under `helm_charts/`.

Valhall uses separate values files for development and production:

```text
values-dev.yaml
values-prod.yaml
```

Render a chart locally before committing:

```bash
helm template <release-name> <chart-directory> -f <values-file>
```

Example:

```bash
helm template bong-valhall \
  helm_charts/backend-valhall/bong-valhall \
  -f helm_charts/backend-valhall/bong-valhall/values-dev.yaml
```

Update Kong chart dependencies when required:

```bash
helm dependency update helm_charts/kong-gateway
```

## MinIO endpoints

MinIO uses separate routes for different trust levels:

```text
Public upload API  -> Cloudflare -> Kong -> MinIO API
Private API        -> Tailscale -> MinIO API
Private console    -> Tailscale -> MinIO console
```

The public route should only expose the bucket or path required for application uploads. The Terraform state bucket should remain reachable only through the private endpoint.

## Secrets

Secrets should not be committed to this repository.

Workloads expect secrets such as:

- Database credentials
- Keycloak configuration
- MinIO credentials
- Cloudflare Tunnel credentials
- TLS Secrets created by cert-manager

Inspect secrets without printing their values:

```bash
kubectl get secrets -A
```

## Common commands

List cluster nodes:

```bash
kubectl get nodes
```

List pods:

```bash
kubectl get pods -A
```

List services:

```bash
kubectl get services -A
```

List ingresses:

```bash
kubectl get ingress -A
```

List namespaces:

```bash
kubectl get namespaces
```

Inspect a resource:

```bash
kubectl describe <resource-type> <resource-name> -n <namespace>
```

View container logs:

```bash
kubectl logs -n <namespace> <pod-name>
```

Follow logs:

```bash
kubectl logs -n <namespace> <pod-name> -f
```

## TLS troubleshooting

Inspect certificates and ACME resources:

```bash
kubectl get certificate,certificaterequest,order,challenge -A
```

Describe a certificate:

```bash
kubectl describe certificate <certificate-name> -n <namespace>
```

Check whether its TLS Secret exists:

```bash
kubectl get secret <tls-secret-name> -n <namespace>
```

A certificate is ready when its condition reports:

```text
Ready: True
```

## RBAC

Kubernetes uses role-based access control to determine what a ServiceAccount may do.

List ServiceAccounts:

```bash
kubectl get serviceaccounts -A
```

Inspect permissions for a ServiceAccount:

```bash
kubectl auth can-i --list \
  --as=system:serviceaccount:<namespace>:<service-account>
```

Applications should receive only the permissions they need.

## GitOps workflow

1. Change a Helm chart or values file.
2. Render or lint the chart locally.
3. Commit and push the change.
4. Allow Argo CD to synchronize it.
5. Verify both sync and health status.
6. Inspect Kubernetes events and logs when health does not become green.

A rollback made only through the Argo CD UI changes the live cluster temporarily. If Git still contains the newer configuration, Argo CD can apply it again. Permanent rollbacks should also be committed to Git.

## App of Apps

This repository uses Argo CD's **App of Apps** pattern. The root application is
defined in `root-application/root.yaml` and watches the `applications/`
directory recursively.

The files inside `applications/` are child Argo CD Applications for services
such as Cloudflare, Kong, Tailscale, MinIO, monitoring, GitHub Actions runners,
and the development and production Valhall environments.

After Argo CD has been installed, bootstrap all child applications by applying
the root application:

```bash
kubectl apply -f root-application/root.yaml
```

Argo CD then creates and synchronizes the child Applications automatically. It
also self-heals changes made directly in the cluster and prunes resources that
are removed from Git.

To add another application, place its Application manifest in `applications/`
and push it to Git. The root application will discover it automatically.

argocd:
create namespace and install
```kubectl create namespace argocd```
```kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml```

Some useful commands:
nodes:
```kubectl get nodes```

pods
```kubectl get pods -A```

services
```kubectl get svc -A```

ingress
```kubectl get ingress -A```

namespaces
```kubectl get namespaces```

# Security
The more I work with kubernetes, the more I learn about how insecure my server have been. 

What is RBAC?
RBAC is stands for Role-based Access Control and it is the kubernetes authorization system. When an application tries to do an actiion then it checks if it has the priveleges to do it. 

It have serviceAccounts which basically says which name it has.

A namespace can have several serviceAccounts. Look at ArgoCD for example. Several pods can ahve the same ServiceAccount which gives them the same permission. If nothing is specified it will get default. 


For each serviceAccounts it has rules which is the permissions it gets to do. 


Every application runs as a serviceAccount in kubernetes. The serviceAccount determuines the privileges for an application. A useful command to see this would be:
```kubectl get serviceaccounts -A```
to see all identities in the cluster node. 


# Security
The more I work with kubernetes, the more I learn about how insecure my server have been. 

What is RBAC?
RBAC is stands for Role-based Access Control and it is the kubernetes authorization system. When an application tries to do an actiion then it checks if it has the priveleges to do it. 

It have serviceAccounts which basically says which name it has.

A namespace can have several serviceAccounts. Look at ArgoCD for example. Several pods can ahve the same ServiceAccount which gives them the same permission. If nothing is specified it will get default. 


For each serviceAccounts it has rules which is the permissions it gets to do. 


Every application runs as a serviceAccount in kubernetes. The serviceAccount determuines the privileges for an application. A useful command to see this would be:
```kubectl get serviceaccounts -A```
to see all identities in the cluster node. 

To see what privileges each namespace have and with service name we have the command
```kubectl auth can-i --list \
--as=system:serviceaccount:namespace:servicename```

```act pull_request -P homeserver-gitops=catthehacker/ubuntu:act-latest```

kong
```helm dependency update helm_charts/kong-gateway```
