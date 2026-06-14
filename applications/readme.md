This is argocd application yaml files. Which contains where argocd should find an application and how to deploy it.

When an application is applied ArgoCD will watch the git repository which was defined in the manifest. If the state of the kubernetes resource differs from the desired state in Git, Then ArgoCD will sinchronize the cluster to match the repository.