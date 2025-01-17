- `istioctl [proxy-status [ps]]` lists all workloads and their synchronization state for every xDS API.
- `istioctl dashboard kiali` to open kiali.
- `istioctl analyze -n <namespace>` analyzes Istio configurations.
- `istioctl x describe pod <pod>` gives you more specific information for a pod.
- `istioctl dashboard envoy deploy/<deployment>` opens a dashboard for that specific deployment sidecar proxy.

```
istioctl proxy-config 
	<cluster|endpoint|listener|route|secret|bootstrap> 
	?[--name <name>]
	?[--fqdn <fqdn>]
	?[--port <port>]
	?[--subset <subset>]
	?[deploy/<deployment>]
	?[-o <json|yaml>]
```

- retrieves proxy config for each of the xDS APIs.
	- `route` shows the port/host and which service that forwards the request to.
		- Example: `*/foo → foo.default.svc`
	- `endpoint` shows the **outbound** hosts that a proxy can see.
		- The cluster in the response has this syntax: `<direction>|<port>|<subset>|<service-name>`. 
		- Example: `foo.default.svc → 1.2.3.4`
	- `cluster` shows the different subsets of a service that a proxy can send requests to.
		- Example: `foo-cluster → foo.default.svc`
	- `listener` shows the port and ip that a envoy proxy is listening for **inbound** requests.
		- Example: `0.0.0.0:8080 → Envoy`

>![[Pasted image 20250115120828.png]]



- If we don't see a cluster (subset) that we were expecting, then that's a sign of misconfiguration.
- The `ENDPOINT` field returned from the `istioctl pc endpoints` is the actual IP of the pod that was discovered by envoy.
### Troubleshooting Walkthrough
This is to find where traffic might be getting stuck and not reaching a particular service.

1. Make sure that the traffic is reaching the gateway using `listeners`.

```bash
istioctl pc listeners \
	deploy/istio-ingressgateway -n istio-system
```

2. Use `routes` to figure out where traffic is going. 

```
istioctl pc routes \
	deploy/istio-ingressgateway -n istio-system \
	--name http.8080 -o yaml
```

By following `routes.route.cluster`, we can see which cluster this virtual host is headed for.

```yaml
  name: "80"
  virtualHosts:
  - domains:
    - catalog.istioinaction.svc.cluster.local
    - catalog
    - catalog.istioinaction.svc
    - catalog.istioinaction
    - 10.96.117.89
    routes:
      route:
        cluster: outbound|80|version-1|catalog.istioinaction.svc.cluster.local
```

3. Using this information, we can look specifically at `version-1` cluster to see which pods it routes to.

```bash
❯ istioctl pc cluster webapp-f479bdc57-862jq --fqdn catalog
SERVICE FQDN                                PORT     SUBSET     DIRECTION     TYPE     DESTINATION RULE
catalog.istioinaction.svc.cluster.local     80       -          outbound      EDS
```

