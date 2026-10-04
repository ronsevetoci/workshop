# KPO workshop setup

The `workshop-kpo` Argo CD Application is intentionally disabled until the
cluster-specific OCI values and IAM prerequisites are supplied.

It combines two sources in one Argo CD Application:

1. The pinned Oracle Karpenter Provider for OCI Helm chart (`1.4.0`).
2. `charts/kpo`, which creates the `OCINodeClass`, `NodePool`, HPA and demo workload.

## 1. Prepare the core node pool

The KPO controller must run on the existing core node pool. The pool already
has the `CriticalAddonsOnly=true:NoSchedule` taint, so add a persistent node
label to that OKE node pool:

```text
workshop.oracle.com/node-pool=core