
- You can inject a sidecar to a yaml file using this `istioctl`. 
	
```bash
    > istioctl kube-inject -f <file>
```

- Running a curl command on a dummy pod.

```bash
$  kubectl run -i -n default --rm --restart=Never dummy \
	--image=curlimages/curl --command -- \
	sh -c 'curl -s http://catalog.istioinaction/items/1'
```

- If you don't specify `gateways` within a `VirtualService`, then the rules apply to traffic originating from the mesh itself.
- You can also specify `mesh` in under `gateways`, if you want it to also apply to internal traffic.
- `subset`'s in `VirtualService` are basically "clusters" in Envoy.