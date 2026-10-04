# OKE Workshop GitOps Repository

Instructor-owned Argo CD App-of-Apps repository for Ron’s OKE workshop topics.

The repository assumes an existing OKE cluster. OCI IAM, dynamic groups, subnets, NSGs, OCI CNI configuration, and load-balancer network rules are external prerequisites.

## Ron’s workshop scope

| Original requirement | Repository mapping | Coverage |
|---|---|---|
| מדיניות אבטחת Pods | `charts/governance/templates/namespace.yaml`, `charts/governance/templates/demo.yaml`, Kyverno application in `apps/values.yaml` | Pod Security Admission and secure workload settings. Kyverno is installed for discussion; no policy demo is included. |
| אופטימיזציית משאבים ועלויות | `charts/kpo/values.yaml`, `charts/kpo/templates/nodepool.yaml` | KPO limits, shape selection, consolidation, and disruption. OCI Cloud Advisor and full workload rightsizing are not implemented here. |
| תשתית ליבה | `bootstrap/root-app.yaml`, `apps/`, `argocd-values.yaml` | Argo CD App-of-Apps and OKE ecosystem deployment. The OKE cluster itself is assumed to exist. |
| ניהול ותחזוקת | `bootstrap/root-app.yaml`, `apps/` | GitOps deployment structure only. OKE maintenance and day-2 operations are covered in the presentation. |
| עדכונים ושדרוגים | No dedicated chart or manifest | Covered in the OKE foundation presentation, not implemented by this repository. |
| ארכיטקטורת High Availability | No dedicated chart or manifest | Covered in the OKE foundation presentation. |
| Cluster Autoscaling | KPO source in `apps/values.yaml`; all files under `charts/kpo/` | KPO `NodePool`, `OCINodeClass`, workload-node taints, secondary VNICs, NSGs, and consolidation. |
| Workload Autoscaling | Metrics Server application in `apps/values.yaml`; all files under `charts/hpa/`; KEDA application in `apps/values.yaml` | CPU-based HPA is demonstrated. KEDA is installed but no `ScaledObject` is included. |
| ניהול והגבלת משאבים (Quotas, Limits & LimitRanges) | `charts/governance/templates/resourcequota.yaml`, `charts/governance/templates/limitrange.yaml` | Namespace-level `ResourceQuota` and `LimitRange`. |
| ניהול חשיפת שירותים ורשת (LB & Ingress) | Envoy Gateway application in `apps/values.yaml`; all files under `charts/ingress/` | Envoy Gateway, OCI Load Balancer annotations, Gateway API, HTTPRoute, and pod-backed demo service. |

## Main repository areas

- `bootstrap/` — bootstrap Application.
- `apps/` — AppProject and child Applications.
- `charts/governance/` — Pod Security, quotas, limits, and secure demo workload.
- `charts/hpa/` — Metrics Server consumer and HPA demonstration workload.
- `charts/kpo/` — KPO NodePool, OCINodeClass, workload, and HPA.
- `charts/ingress/` — Envoy Gateway and OCI Load Balancer configuration.
