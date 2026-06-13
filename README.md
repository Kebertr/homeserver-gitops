# homeserver-gitops
Gitops for the applicatiions in my homeserver

When added a new application. Run
```kubectl apply -f applications/[name].yaml```

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

Cloudflare tunnel:
Creating:
```cloudflared tunnel run [name]```

login to Cloudflare:
```cloudflared tunnel login```

run it:
```cloudflared tunnel run homelab```
