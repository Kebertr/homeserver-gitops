# llm-kg

This chart deploys image `0.1.0` with environment variables from the existing
Kubernetes Secret `llm-kg-env`. It does not create or commit credentials.
The Argo CD Application syncs manually. No GraphDB instance, public route, or
image update watcher is provisioned yet. Confirm the application's runtime
(long-running service versus job), listening port, and GraphDB image/version
before enabling deployment and automatic image updates.

## Environment Secret

Create namespace `llm-kg` if it does not already exist. Create a local file
`secrets/llm-kg.env` (the root `secrets` directory is ignored by Git) containing:

```dotenv
GRAPHDB_URL=http://graphdb:7200/repositories/Pizza_60
GRAPHDB_DOCKER_URL=http://graphdb:7200/repositories/Pizza_60
LITELLM_AGENT_URL=https://your-llm-service/v1/chat/completions
LITELLM_API_KEY=replace-with-your-api-key
LITELLM_MODEL=replace-with-your-model-name
```

Replace the LiteLLM placeholders with real values. Use plain `KEY=value` lines,
without escaped colons, trailing backslashes, or surrounding quotes.
Both GraphDB URLs assume a Service named `graphdb` in namespace `llm-kg`.
For another namespace use `graphdb.<namespace>.svc.cluster.local` instead.

```bash
kubectl create namespace llm-kg --dry-run=client -o yaml | kubectl apply -f -
chmod 600 secrets/llm-kg.env
kubectl -n llm-kg create secret generic llm-kg-env \
  --from-env-file=secrets/llm-kg.env --dry-run=client -o yaml |
  kubectl apply -f -
```

Also provision the private registry Secret `ghcr-pull-secret` in `llm-kg`.
A registry Secret in another namespace is not usable by this Deployment.
After updating environment values, restart the application to load them:

```bash
kubectl -n llm-kg rollout restart deployment/llm-kg
```

Secrets belong in Kubernetes Secrets, not ConfigMaps. If an existing secret
manager is used, have it populate `llm-kg-env` with these five keys instead.

## GraphDB repositories

GraphDB still needs to be deployed with persistent storage and an internal
Service. Once the `graphdb` Service exists in namespace `llm-kg`, access its
Workbench locally:

```bash
kubectl -n llm-kg port-forward service/graphdb 7200:7200
```

Open http://localhost:7200, log in if security is enabled, then use
**Setup → Repositories → Create new repository**. Use repository ID `Pizza_60`
(case-sensitive), choose the required ruleset, and save. Select that repository
and use **Import → Upload RDF files** to import your ontology/data. A repository
configuration `.ttl` and an RDF data `.ttl` serve different purposes.

If you already have an exported repository configuration, create it through
the REST API instead (add authentication if required by your instance):

```bash
curl --fail-with-body -X POST http://localhost:7200/rest/repositories \
  -F 'config=@repo-config.ttl'
```

See [GraphDB repository API documentation](https://graphdb.ontotext.com/documentation/11.4/manage-repos-with-restapi.html).
An existing populated repository should be transferred using GraphDB's supported
export/import or backup/restore tools for your version. Repository creation alone
does not copy data. Keep GraphDB data on a PVC so pod replacement preserves it.
