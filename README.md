# OKE Workshop GitOps Repository

Instructor-owned Argo CD App-of-Apps repository for Ron’s OKE workshop topics.

The repository assumes an existing OKE cluster. OCI IAM, dynamic groups, subnets, NSGs, OCI CNI configuration, and load-balancer network rules are external prerequisites.

## Scope
https://kubernetes.io/docs/concepts/security/pod-security-admission/

## Main repository areas

- `bootstrap/` — bootstrap Application.
- `apps/` — AppProject and child Applications.
- `charts/governance/` — Pod Security, quotas, and limits.
- `charts/kpo/` — KPO NodePool and OCINodeClass.
- `charts/ingress/` — Envoy Gateway and OCI Load Balancer configuration.
- `charts/demo/` — all workshop sample workloads, HPAs, services, and the demo HTTPRoute. One Argo CD Application owns them across their existing namespaces.

## Demo application

`workshop-demo` is the only Argo CD Application that owns sample workloads. It deploys `hpa-demo` to `workshop-hpa`, `kpo-scale-demo` to `workshop-kpo`, `governance-demo` to `workshop-governance`, and `demo-app` plus its route to `envoy-gateway-system`. The existing names and namespaces remain the same. The KPO and governance applications still own the namespaces and infrastructure those demos depend on; the demo application owns the `workshop-hpa` namespace because the old HPA application contained only a sample workload. The demo Application has sync wave 50, after its dependencies.

Configure sample images, replicas, scaling, and placement in `charts/demo/values.yaml`. When enabling the ingress HTTPS listener in `charts/ingress/values.yaml`, also enable `ingress.tls.enabled` in `charts/demo/values.yaml` so the demo route attaches to it.

### Existing cluster handoff

These resources previously belonged to four Argo CD Applications. Before syncing the changed root application with pruning, remove the deletion finalizer from the existing `workshop-hpa` Application so its removal does not cascade-delete `workshop-hpa`:

```sh
kubectl -n argocd patch application workshop-hpa --type=merge -p '{"metadata":{"finalizers":[]}}'
```

Then sync the root application, sync the KPO, governance, and ingress applications without pruning, and sync `workshop-demo` to transfer resource tracking. Confirm the demo Application owns all 12 sample resources and is healthy before pruning obsolete resources from the former owners. Keep the KPO, governance, and ingress Applications and their namespaces.
