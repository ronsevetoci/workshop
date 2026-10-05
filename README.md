# OKE Workshop GitOps Repository

Instructor-owned Argo CD App-of-Apps repository for Ron’s OKE workshop topics.

The repository assumes an existing OKE cluster. OCI IAM, dynamic groups, subnets, NSGs, OCI CNI configuration, and load-balancer network rules are external prerequisites.

## Ron’s workshop scope

| Original requirement | Repository mapping | Coverage |
|---|---|---|
| מדיניות אבטחת Pods | `charts/governance/templates/namespace.yaml`, `charts/demo/templates/governance.yaml`, Kyverno application in `apps/values.yaml` | Pod Security Admission and secure workload settings. Kyverno is installed for discussion; no policy demo is included. |
| אופטימיזציית משאבים ועלויות | `charts/kpo/values.yaml`, `charts/kpo/templates/nodepool.yaml` | KPO limits, shape selection, consolidation, and disruption. OCI Cloud Advisor and full workload rightsizing are not implemented here. |
| תשתית ליבה | `bootstrap/root-app.yaml`, `apps/`, `argocd-values.yaml` | Argo CD App-of-Apps and OKE ecosystem deployment. The OKE cluster itself is assumed to exist. |
| ניהול ותחזוקת | `bootstrap/root-app.yaml`, `apps/` | GitOps deployment structure only. OKE maintenance and day-2 operations are covered in the presentation. |
| עדכונים ושדרוגים | No dedicated chart or manifest | Covered in the OKE foundation presentation, not implemented by this repository. |
| ארכיטקטורת High Availability | No dedicated chart or manifest | Covered in the OKE foundation presentation. |
| Cluster Autoscaling | KPO source in `apps/values.yaml`; `charts/kpo/`; `charts/demo/templates/kpo-*` | KPO `NodePool`, `OCINodeClass`, workload-node taints, secondary VNICs, NSGs, and consolidation. The sample workload lives in the demo application. |
| Workload Autoscaling | Metrics Server and KEDA applications in `apps/values.yaml`; `charts/demo/templates/hpa-*` | CPU-based HPA is demonstrated by the demo application. KEDA is installed but no `ScaledObject` is included. |
| ניהול והגבלת משאבים (Quotas, Limits & LimitRanges) | `charts/governance/templates/resourcequota.yaml`, `charts/governance/templates/limitrange.yaml` | Namespace-level `ResourceQuota` and `LimitRange`. |
| ניהול חשיפת שירותים ורשת (LB & Ingress) | Envoy Gateway application in `apps/values.yaml`; `charts/ingress/`; `charts/demo/templates/ingress-*` | Envoy Gateway, OCI Load Balancer annotations, Gateway API, and a demo-owned HTTPRoute and service. |

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
