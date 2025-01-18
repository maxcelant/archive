- **Latency** is measured by how quickly the control plane dispatches updates to the data plane.
- **Saturation** occurs when the queue becomes backed up with updates and they can't be pushed out fast enough.
- `Sidecar` resource is used to restrict what services proxies can see. It also reduces the overall "push count".
- It would be smart to configure them to only see the `istio-system` and any observation tools by default.

```yaml
apiVersion: networking.istio.io/v1beta1
kind: Sidecar
metadata:
  name: default
  namespace: istioinaction
spec:
  workloadSelector:
    labels:
      app: foo
  egress:
  - hosts:
    - "./bar.istioinaction.svc.cluster.local"
    - "istio-system/*"
```

- You can see how much data the config size takes up. Roughly 2 MB per workload.

```bash
$ k exec -ti <pod> -c <main-app> \
	-- curl -s localhost:15000/config_dump > /tmp/config_dump

$ du -sh /tmp/config_dump
```

- You can add "discovery selectors" to your `IstioOperator` which tells istio which workloads it should reconcile and which it shouldn't.
- 