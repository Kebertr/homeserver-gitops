# Architecture

## System overview

### Simplified diagram for system overview
```mermaid
flowchart LR
    admin[Administrator]
    users[Public users]

    subgraph github[GitHub]
        source[Valhall source]
        actions[GitHub Actions]
        registry[Container images]
        gitops[GitOps configuration]
    end

    cloudflare[Cloudflare]
    tailscale[Tailscale]

    subgraph cluster[K3s cluster]
        argocd[Argo CD]
        networking[Kong and Traefik]
        valhall[Valhall<br/>development and production]
        keycloak[Keycloak]
        monitoring[Grafana and Prometheus]
        storage[(PostgreSQL and MinIO)]
    end

    admin -->|push code| source
    source --> actions
    actions -->|publish| registry

    registry -->|image updates| gitops
    argocd -->|read desired state| gitops
    argocd -->|deploy and synchronize| valhall

    users --> cloudflare
    cloudflare --> networking
    networking --> valhall
    networking --> keycloak

    admin --> tailscale
    tailscale --> argocd
    tailscale --> monitoring
    tailscale --> storage

    valhall --> keycloak
    valhall --> storage
    monitoring --> valhall
```

## Cluster architecture
```mermaid
flowchart TB
    subgraph cluster[K3s cluster]
        subgraph servers[Three K3s server nodes]
            scheduler[Kubernetes scheduler<br/>runs in the control plane]

            subgraph server1[Server 1: control plane, workloads and storage]
                server1Services[etcd member<br/>stateless API workloads]
                postgres[PostgreSQL databases]
                minio[MinIO]
                disk[(Local persistent storage)]

                postgres -->|database data| disk
                minio -->|object data| disk
            end

            server2[Server 2<br/>etcd member<br/>stateless API workloads]
            server3[Server 3<br/>etcd member<br/>stateless API workloads]

            server1Services <-->|etcd replication| server2
            server2 <-->|etcd replication| server3
            server3 <-->|etcd replication| server1Services
        end

        subgraph agents[Agent nodes]
            agent1[Agent 1<br/>stateless API workloads]
        end
    end

    scheduler -.->|may schedule APIs| server1Services
    scheduler -.->|may schedule APIs| server2
    scheduler -.->|may schedule APIs| server3
    scheduler -.->|may schedule APIs| agent1

    scheduler -->|stateful workloads pinned to Server 1| postgres
    scheduler -->|stateful workloads pinned to Server 1| minio
```

This diagram demonstrates how the clsuter work. We have acontrol plane of 3 nodes. Then I have one worker that's not reliably active. So we need to store replicas on one of the 3 nodes in the control plane aswell.

The storage right now is only on server 1. This have disadvantages and will be updated in teh future with wal and more. I have already started implementing cloudNativePg for this. But right now the stateful workloads need to be on server 1 since it is there where they are stored on the physical disk. the other 2 servers in the control plane can also take stateless API:s. There is a etch replication and snapshot on all. 

## Gitops architecture

## Network architecture

## Valhall application architecutre

## Storage architecture