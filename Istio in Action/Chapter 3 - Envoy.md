
- Envoy is extendable through its filters.
- Envoy adds a bunch of headers to traffic requests in order to track them as they flow to gain tracing capabilities.

```json
	{
		"headers": {
			"Accept": "*/*",  
			"Content-Length": "0",  
			"Host": "httpbin",  
			"User-Agent": "curl/7.80.0", 
			"X-Envoy-Expected-Rq-Timeout-Ms": "15000", 
			"X-Request-Id": "45f74d49-7933-4077-b315-c15183d1da90"
		} 
	}
```

- **listeners**: tell envoy on what ip and port to listen to traffic on. 
- **filters**: do _something_ to that traffic 
- **clusters**: tell envoy which pool of services to send the traffic to.

```yaml
static_resources:
  listeners:
    - name: httpbin-demo
      address:
        socket_address: 
          address: 0.0.0.0
          port_value: 15001
      filter_chains:
        - filters:
            - name: envoy.http_connection_manager
              config:
                stat_prefix: egress_http
                route_config:
                  name: httpbin_local_route
                  virtual_hosts:
                    - name: httpbin_local_service
                      domains: ["*"]  # Wildcard virtual hosts
                      routes:
                        - match: 
                            prefix: "/"
                          route:
                            auto_host_rewrite: true
                            cluster: httpbin_service
                http_filters:
                  - name: envoy.router
  clusters:
    - name: httpbin_service
      connect_timeout: 5s
      type: LOGICAL_DNS
      # Comment out the following line to test on v6 networks
      dns_lookup_family: V4_ONLY
      lb_policy: ROUND_ROBIN
      hosts:
        - socket_address:
            address: httpbin
            port_value: 8000

```

- Envoy has a `/stats` endpoint which has a bunch of counters for different events like retries, changes, errors, etc.

```bash
❯ docker run -it --rm --link proxy curlimages/curl \
curl -X GET http://proxy:15000/stats

❯ k exec -it deploy/foobar -c istio-proxy \ 
-- curl localhost:15000/stats
```

- There's also `/listeners` if you want to know which IP/ports it's listening on.
- Envoy has a suite of "dynamic runtime configurations" which it calls the _xDS API_. This is implemented using gRPC. This allows Istio to update its config without restarts.
- `istiod` reads kubernetes resources (like virtual services) and configures the envoy sidecars using this xDS API.
- `istiod` uses Kubernetes native service discovery, leveraging the `Endpoints` object and translates that to Envoy EDS (Endpoint Discovery Service).