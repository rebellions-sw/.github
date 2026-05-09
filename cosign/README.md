# Cosign signing key

`cosign.pub` is the **public** verification key for all RBCN-built container images. The corresponding **private** key lives in GitHub Org secrets as `RBCN_COSIGN_PRIVATE_KEY` (and password in `RBCN_COSIGN_PASSWORD`), used by `rbcn-github-actions/.github/workflows/build-app.yml` at sign time.

## Verifying an image manually

```bash
cosign verify --key cosign.pub harbor.infra.rblnconnect.ai/rbcn/demo-api:v0.0.1-dev.42
```

## How it gets enforced in clusters

The `admission/cluster-image-policy-rbcn.yaml` (this repo) embeds this public key inline. Sigstore policy-controller (deployed via `rbcn-iac-ansible/roles/k8s_policy_controller`) consumes it on stage + prod clusters and rejects pod admissions with `harbor.infra.rblnconnect.ai/rbcn/*` images that don't carry a valid Cosign signature against this key.

dev cluster is intentionally **not** enforced (faster iteration).

## Rotation

1. `cosign generate-key-pair` (new keypair)
2. Update GitHub Org secrets `RBCN_COSIGN_PRIVATE_KEY` + `RBCN_COSIGN_PASSWORD`
3. Commit new `cosign/cosign.pub` to dev branch (this repo)
4. Re-run ansible policy-controller playbook to push new ClusterImagePolicy to stage + prod
5. **All currently signed images stop verifying** — kick a fresh build of every app to re-sign with new key. During the gap, stage/prod admission will reject deploys.

For seamless rotation: support multiple authorities in the ClusterImagePolicy temporarily (old + new), wait for old sigs to age out, then drop the old authority.
