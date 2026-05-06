# rbcn-k8s-policies

Kyverno ClusterPolicies enforced on all RBCN K8s clusters.

## Policies
| Policy | Enforcement |
|---|---|
| require-signed-image | Cosign signed only (Harbor + sigstore-cosign keyless) |
| disallow-host-path | hostPath volumes blocked |
| disallow-host-network | hostNetwork blocked |
| require-run-as-non-root | runAsNonRoot=true required |
| require-image-pull-secrets | imagePullSecrets required |
| require-resource-limits | requests + limits required |
| require-pdb | PodDisruptionBudget required for Deployments |

## Source
Synced by rbcn-k8s-platform → applications/kyverno-policies.yaml

## Owner
seanlee@rebellions.ai
