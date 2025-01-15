- For **Least Request** (`LEAST_CONN`) load balancing, Envoy is randomly looking at two endpoints and choosing the one with fewer active requests.
- **Fortio** is a good tool for load testing your services.
- When deploying kubernetes, labels can be added to the nodes for information about region and zone: `topology.kubernetes .io/region` and `topology.kubernetes.io/zone`. These will be automatically added by Azure.
- Locality-aware load balancing is enabled by default.
- We can add passive health-checking by configuring outlier detection in a `DestinationRule`. Without it, locality-aware load balancing won't work! 
	- This is the case because Istio needs to know which endpoints are healthy and which aren't. 
	- If the endpoints in the preferred locality are unhealthy or overwhelmed, Istio needs a heuristic to decide when to "spill over" traffic to other localities.

```yaml
apiVersion: networking.istio.io/v1beta1
kind: DestinationRule
metadata:
  name: simple-backend-dr
spec:
  host: simple-backend.istioinaction.svc.cluster.local 
  trafficPolicy:
    connectionPool:
      http:
        http2MaxRequests: 10
        maxRequestsPerConnection: 10
    outlierDetection:
      consecutiveErrors: 1
      interval: 1m
      baseEjectionTime: 30s
```

- We can spill over a percentage of the traffic coming from a zone/region to a different zone/region.

```yaml
apiVersion: networking.istio.io/v1beta1
kind: DestinationRule
metadata:
  name: simple-backend-dr
spec:
  host: simple-backend.istioinaction.svc.cluster.local 
  trafficPolicy:
    loadbalancer:
      localityLbSetting:
        distribute:
        - from: us-west1/us-west1-a/*
          to:
            "us-west/us-west1-a/*": 70
            "us-west/us-west1-b/*": 30
    outlierDetection:
      consecutiveErrors: 1
      interval: 1m
      baseEjectionTime: 30s
```

- You can customize load balancing rules per subset within your `DestionationRule`.
- Generally, it makes sense to have larger timeouts at the edge (where traffic comes in) of an architecture and shorter (or more restrictive) timeouts for the layers deeper in the call graph.
- You can implement timeouts, retries in a `VirtualService` config.
- You can configure specific errors to retry on, the timeout time per retry and even whether to retry endpoints in other localities.
- `perTryTimeout` value multiplied by the total number of attempts must be less than the overall request timeout.
- If you set retries to 2, then it will be sent up to 3 times in total.
- The `EnvoyFilter` API is a "break glass" solution that allows you to configure/override Envoy's behavior.
- **Circuit breaking** limits the number of connections or requests, and when those thresholds are exceeded, Envoy returns 503 errors (not 500s) for the excess
- When a request times out, Envoy sends that same request to a different endpoint to "race" the original, if that one succeeds then the response is sent downstream to the original caller. This is called **Request Hedging** and it is configurable through `EnvoyFilter` API.
- `connectionPool` setting in the `DestinationRule` is used to limit the number of connections and requests that can pile up when calling a service.
- Here's an example of using `fortio` so load test an application

```bash
$ fortio load -H "Host: simple-web.istioinaction.io" \ -quiet -jitter -t 30s -c 2 -qps 2 http://localhost/
```

- When a request fails for tripping a circuit-breaking threshold, Istio’s service proxy adds an `x-envoy-overloaded` header. 
- **Outlier detection** is helpful because there may be times in which your health endpoint is working properly but specific service endpoints are not. So kubernetes doesn't know something is wrong.

```yaml
apiVersion: networking.istio.io/v1beta1
kind: DestinationRule
metadata:
  name: simple-backend-dr
spec:
host: simple-backend.istioinaction.svc.cluster.local trafficPolicy:
    outlierDetection:
      consecutive5xxErrors: 1
      interval: 5s
      baseEjectionTime: 5s
      maxEjectionPercent: 100
```
