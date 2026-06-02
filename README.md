# homeserver-gitops
Gitops for the applicatiions in my homeserver

When added a new application. Run
```kubectl apply -f applications/[name].yaml```

argocd:
create namespace and install
```kubectl create namespace argocd```
```kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml```
