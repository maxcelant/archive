- You can have different hostnames `prod.foo.io` and `dev.foo.io` resolve to the same virtual IP (ingress point) and then use the HTTP `Host` to further delineate (literally [[SNI]]). This is called _virtual hosting_.
- We use virtual services to define how clients talk to specific services using [[FQDN]], which versions of service are available, and other routing properties.
- Virtual services also define what to do with traffic when it comes into a gateway.
- This command lets us see the "routes" attached to a specific envoy proxy.

```bash
$ istioctl pc route deploy/istio-ingressgateway \
  -o json --name http.8080 -n istio-system
```
- In order for your `curl` command to work, you need to use the host that you set in the gateway and vs. For example if your gateway and vs are set with `max.teaching.io`, then you need to do this:

```bash
$ curl http://localhost:8080/api/catalog -H "Host: max.teaching.io"
```

- A certificate is just a public key that has been signed by a reputable authority (CA). 
- We need to use `--resolve` when making a TLS/mTLS `curl` command because the certificate is issued for that specific hostname, not localhost. But we're calling it with localhost so we `--resolve` overrides that.
- In more detail `--resolve` overrides the DNS lookup and uses whatever you resolve it to with the syntax `hostname:port:ip`. This is only necessary when you aren't port-forwarding the ingress gateway.

```bash
$ curl -H "Host: webapp.istioinaction.io" \ 
   https://webapp.istioinaction.io:443/api/catalog \      
   --cacert ch4/certs/2_intermediate/certs/ca-chain.cert.pem \ 
   --resolve webapp.istioinaction.io:443:127.0.0.1
```

- To perform an mTLS request using `curl`, you need to also send the `key` and `cert` because you are also verifying yourself, just like the server does normally in TLS.

```bash
$ curl -H "Host: webapp.istioinaction.io" \ https://webapp.istioinaction.io:443/api/catalog \
--cacert ch4/certs/2_intermediate/certs/ca-chain.cert.pem \
--resolve webapp.istioinaction.io:443:127.0.0.1 \
--cert ch4/certs/4_client/certs/webapp.istioinaction.io.cert.pem \ 
--key ch4/certs/4_client/private/webapp.istioinaction.io.key.pem
```