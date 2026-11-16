# Minecraft with Crafty Controller

Crafty runs the dashboard and Minecraft process together on `keb-k3s`.
The container has a combined 10Gi memory request/limit. Set the Minecraft
maximum Java heap to **8192 MiB (8 GiB)** in Crafty, leaving room for Java's
other allocations and the panel. This is one shared limit for all servers
created in Crafty, not 10Gi per server. Start with one server and 20 player slots.

## Hard drive storage

Data is stored in `/mnt/valhall-media/minecraft` on `keb-k3s`, separately from
MinIO's `/mnt/valhall-media/minio`. No local-path PVC is created by this chart.
The parent `/mnt/valhall-media` must already exist. The init container creates
only Minecraft's config, servers, backups, logs and import subdirectories and
sets their ownership to Crafty's UID 1000, group 0; it does not change MinIO.

Before deploying, run on **keb-k3s**:

```bash
findmnt --target /mnt/valhall-media
 df -h /mnt/valhall-media
```

Confirm that this is the intended hard drive, with adequate free space. The
chart checks that the parent directory exists, but cannot prove the drive is
mounted. Ensure the disk mounts persistently before K3s starts, including after
reboots; otherwise data could be written to the underlying system disk.
The hostname label must be `keb-k3s`. The node needs 10Gi memory available for
scheduling after other workloads' requests.

## Deploy and open the dashboard

Commit and push the chart and `applications/minecraft-app.yaml`; the root Argo
CD Application discovers it. The dashboard starts immediately; it does not
create a Minecraft server or accept the Minecraft EULA automatically.

```bash
kubectl -n minecraft get pods,svc
kubectl -n minecraft logs -f statefulset/minecraft-crafty
```

With the existing Tailscale operator running, open **https://minecraft-admin**
from a device connected to your tailnet (or use its full MagicDNS name).
Tailnet access policy must allow access. The dashboard service exposes only
HTTPS through Tailscale; do not forward its port on your router.
This is TCP forwarding to Crafty's own HTTPS endpoint, so its default
self-signed certificate will produce a browser certificate warning. Verify the
address before proceeding, or install a trusted certificate in Crafty.

Read the generated initial credentials privately in your terminal:

```bash
kubectl -n minecraft exec statefulset/minecraft-crafty -- cat /crafty/app/config/default-creds.txt
```

Log in and change the generated password. Do not commit credentials to Git.
For temporary access without Tailscale:

```bash
kubectl -n minecraft port-forward service/minecraft-dashboard 8443:443
```

Then open https://localhost:8443 (also uses Crafty's certificate).

## Create the Minecraft server in Crafty

1. Create a Java server, selecting your desired version or mod loader. Ensure
   the bundled Java runtime supports the version you choose.
2. Configure its game port as **25565**, bound to **0.0.0.0**. This chart exposes
   only that game port; extra servers or Dynmap need additional Services.
3. Set maximum heap to **8192 MiB**, and maximum players to **20**. Keep the
   initial heap smaller if you want the stopped/idle server to use less memory.
4. Read and accept the Minecraft EULA yourself when setting up the server.
5. Keep `online-mode=true`, set `white-list=true` and `enforce-whitelist=true`,
   and leave RCON and query disabled. Start it, then use Crafty's console:

```text
whitelist add YourMinecraftUsername
whitelist list
```

The whitelist can also be edited through Crafty's server file tools. Use the
console commands for changes while running. Configure scheduled world backups
in Crafty and enable automatic server startup if desired. Crafty's UI controls
Minecraft; Kubernetes/Argo CD controls the panel container itself.

## Public game access

Reserve a stable LAN IP for the K3s node. Forward **TCP 25565** on your router
to **keb-k3s LAN IP:30565** and allow it through the host firewall. Do not
forward dashboard, SSH, Kubernetes API, database ports, or all NodePorts.

Create a Cloudflare A record `mc.kebert.se` pointing at your public IPv4,
**DNS only** (grey cloud). Players join `mc.kebert.se`. There is no paid
Cloudflare feature or Cloudflare DDoS protection here. Update DNS if the IP
changes. Test from outside your home network; locally use `NODE_LAN_IP:30565`
if NAT loopback is unavailable. Configure IPv6 separately before adding AAAA.

## Backups, upgrades and existing data

Crafty's world backups reside on the same hard drive; copy them to separate
storage for protection against disk failure. Also back up its config/database
and other data directories while Crafty is stopped (`replicas: 0` in Git,
wait for termination before copying). Restore `replicas: 1` afterward.
Pin `image` to a tested release tag or digest before routine production use;
the initial `latest` tag can change on pod recreation. Back up before upgrades.

This replaces the earlier standalone Minecraft deployment. It does not import
an existing world automatically. If that deployment was already used, stop it
and preserve its `world-minecraft-0` PVC, export the world, and import through
Crafty before deleting old storage. The new StatefulSet is `minecraft-crafty`.

References:
- https://docs.craftycontrol.com/pages/getting-started/installation/docker/
- https://docs.craftycontrol.com/pages/getting-started/access/
- https://tailscale.com/docs/kubernetes-operator/ingress/expose-workload-to-tailnet-l3
