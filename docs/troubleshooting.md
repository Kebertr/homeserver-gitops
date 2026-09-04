# Troubleshooting

## Nodes
Get all nodes
```bash
kubectl get nodes
```

```bash
kubectl describe node <node-name>
```

Also visualizing every node in grafana. Check there.

Get all nodes with more information
```bash 
kubectl get nodes -o wide
```

## Pods
Get all pods in the cluster
```bash
kubectl get pods -A
```

or if it's a special namespace 

```bash
kubectl get pods -n <namespace>
```

```bash
kubectl describe pods <pod-name> -n <namespace>
```

View container logs:

```bash
kubectl logs -n <namespace> <pod-name>
```

Follow logs:

```bash
kubectl logs -n <namespace> <pod-name> -f
```

## secrets
```bash 
kubectl get secrets -n <namespace>
```

Check if the secret have something in it
```bash
kubectl describe secret <secret-name> -n <namespace>
```

## Services
```bash
kubectl get services -n <namespace>
```

```bash 
kubectl get endpoints -n <namespace>
```

```bash
kubectl describe service <service-name> -n <namespace>
```

## ingress
This I use quite often to get the ingress of my internal sensitive tools lika prometheus, argo and so on.
```bash
kubectl get ingress -n <namespace>
```

```bash
kubectl describe ingress <ingress-name> -n <namespace>
```

```bash
kubectl get ingressClass
```

## Storage
For storage you could call for something like 

```bash
kubectl get pvc -n <namespace>
```

```bash
kubectl get pv
```

```bash
kubectl describe pvc <pvc-name> -n <namespace>
```

```bash
kubectl describe pv <pv-name>
```

But I prefer to connect to all databases with a third party tool like dbeaver

## Argo CD
I like the UI for this very much. Some times it might be hard to navigate

```bash
kubectl get applications -n argocd
```

```bash
kubectl describe application <application-name> -n argocd
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

## Snapshot
```bash
kubectl get volumesnapshotclass
kubectl get volumesnapshots -A
kubectl get volumesnapshotcontents
```

## How to know which to use?
It can be very hard to know what to run and what to check when something goes down. I usually start with checking the pods for what goes down. See if they are up and also check their logs to see what might be wrong. That is how you often can solve things. Other times if it is problem with communicating internally I would recommend seeing over service. If you can't reach them from the external internet I would start with ingress.