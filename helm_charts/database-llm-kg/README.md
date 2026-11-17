# GraphDB for llm-kg

Follows Valhall's database pattern: separate Argo CD Application, one StatefulSet
replica, and a 20Gi ReadWriteOnce PVC using `local-path`. The vendor image is
`ontotext/graphdb:11.5.0`; it does not use the application's GHCR credentials.
Data and GraphDB configuration persist under `/opt/graphdb/home`.

Push the manifests to main and sync/discover Application `database-llm-kg`
using your existing applications workflow. If applications are registered
manually, run `kubectl apply -f applications/llm-kg-db.yaml` after pushing.

```bash
kubectl -n llm-kg rollout status statefulset/graphdb --timeout=10m
kubectl -n llm-kg get pods,pvc,svc
kubectl -n llm-kg port-forward service/graphdb 7200:7200
```

Run port-forward on your browser's machine with cluster access. If using SSH,
forward its local port too: `ssh -L 7200:127.0.0.1:7200 keb@192.168.1.100`,
then run the kubectl port-forward inside that SSH session.

Open http://localhost:7200. Under Setup → Repositories create `Pizza_60`, select
the ruleset your application expects, and import the ontology/data through Import.
You can import an exported repository configuration instead if you have one.
Creation does not automatically import your existing local GraphDB data.

Both keys `GRAPHDB_URL` and `GRAPHDB_DOCKER_URL` in `llm-kg-env` should be
`http://graphdb:7200/repositories/Pizza_60`. If changing the Secret, restart
Deployment `llm-kg` to reload its environment variables.

The Service is internal only; GraphDB is not routed through Cloudflare.
No GraphDB credentials are provisioned by this chart. If you enable GraphDB
authentication, configure matching client authentication in the application.

The initial heap is 1Gi with a 4Gi container memory limit to leave room for
off-heap usage. Tune memory to the dataset and workload. TCP probes confirm
the listening port, not repository or query health.

As with Valhall's local-path databases, storage is tied to its selected node.
Pod replacement retains data, but this is not replicated storage or a backup.
Do not delete the PVC unless intentionally deleting the database; back up data
before node maintenance or storage changes.
