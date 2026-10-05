# KPO workshop setup

The `workshop-kpo` Argo CD Application is enabled in `apps/values.yaml`. It combines
the Oracle Karpenter Provider for OCI Helm chart (`1.4.0`) with `charts/kpo`, which
creates the `workshop-kpo` namespace, `OCINodeClass`, and `NodePool`.

The KPO controller runs on the existing `np-system` node pool. Its node selector is:

```yaml
oke.oraclecloud.com/pool.name: np-system
```

It tolerates the system-pool taint `CriticalAddonsOnly=true:NoSchedule`:

```yaml
- key: CriticalAddonsOnly
  operator: Equal
  value: "true"
  effect: NoSchedule
```

The `kpo-scale-demo` workload and its HPA are owned by `workshop-demo` in
`charts/demo/`. The workload targets the KPO-managed `workshop-kpo-apps` NodePool.
IAM, dynamic-group `CLUSTER_JOIN`, OCI CNI, worker and pod subnets, and NSG
prerequisites are external to this repository.
