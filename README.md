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

