# homeserver-gitops
Gitops for the applicatiions in my homeserver

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

To see what privileges each namespace have and with service name we have the command
```kubectl auth can-i --list \
--as=system:serviceaccount:namespace:servicename```

```act pull_request -P homeserver-gitops=catthehacker/ubuntu:act-latest```

kong
```helm dependency update helm_charts/kong-gateway```
