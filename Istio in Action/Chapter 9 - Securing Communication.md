- `PeerAuthentication` policies apply to the whole cluster. They must be named "default" if we want them to be mesh-wide.
- `spec.mtls.mode` set to `PERMISSIVE` allows both plaintext and encrypted traffic.
- `istiod` transforms the `PeerAuthentication` resource into a configuration that Envoy understands and applies it using its **listener discovery service (LDS)**. 
- We can use a `sleep` deployment to make requests from a different namespace to our indicated workload. This similar to what we did in [[Chapter 2 - First steps with Istio]]. 

```bash
k -n default exec deploy/sleep -c sleep -- \
	curl -s webapp.istioinaction/api/catalog \
  -o /dev/null -w "%{http_code}"
```

- You can update the the privileges of the envoy proxies to perform a `tcpdump`. You should only update the privileges through a `kubectl edit` and not actually ship it with these heightened privileges.
- You can use `openssl` to check the X.509 certificate / SVID document of the (catalog) service that had a TLS handshake with the (webapp) service. It'll also contain the [[SPIFFE|SPIFEE ID]]

```bash
kubectl -n istioinaction exec deploy/webapp -c istio-proxy \
     -- openssl s_client -showcerts \
     -connect catalog.istioinaction.svc.cluster.local:80 \
     -CAfile /var/run/secrets/istio/root-cert.pem | \
     openssl x509 -in /dev/stdin -text -noout
```

- `AuthorizationPolicy` resources limit what a resource can do. Like who it can talk to.
	- You can authorize to only have traffic from specific SPIFFE IDs through the `principals` field.

```yaml
apiVersion: security.istio.io/v1 
kind: AuthorizationPolicy 
metadata: 
  name: restrict-traffic-to-red 
  namespace: red 
spec: 
  action: ALLOW 
  rules: 
    - from: 
        - source: 
            principals: ["spiffe://<TRUST_DOMAIN>/ns/prism/sa/default"] 
```

- If one or more `ALLOW` authorization policies are applied to a workload, access will be denied by default for all traffic. This is called _deny by default_.
- There's a `jwt` cli to decode tokens

```bash
echo $jwt_token | jwt decode -
```

### Aside about JSON Web Tokens

- A JWT looks like this:

```
<header>.<payload>.<signature>
```

- The `signature` is a hash of the header and payload, signed using the issuer's private key.
- This means that any tampering with the token will cause a different `signature`, since they don't know the private key.
- Think of this a readonly key. Any changes won't work.
- Receivers use the issuers public key to verify that it wasn't tampered with. 