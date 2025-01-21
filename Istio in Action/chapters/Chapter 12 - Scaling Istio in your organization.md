- You can configure "SNI passthrough" on your east-west gateways. What that means is that the gateway won't decrypt the traffic request and re-route it to the destination. It will just use the SNI packet header to figure out where it should go. 
	- `CLUSTER: outbound_.80_._.catalog.default.svc.cluster.local`

```yaml
apiVersion: networking.istio.io/v1beta1
kind: Gateway
metadata:
  name: east-west-gateway
  namespace: istio-system
spec:
  selector:
    istio: eastwestgateway # Use the appropriate label for your gateway
  servers:
  - port:
      number: 15443
      name: tls
      protocol: TLS
    hosts:
    - "*.local" # Adjust this to match your service domain
    tls:
      mode: AUTO_PASSTHROUGH
```

- By default, istiod acts as a root CA by storing the root cert and private key in-memory (for security). It then signs leaf certs and sends them to the request workloads.
- As a reminder, A Certificate Signing Request (CSR) contains things like the common name (`example.com`), public key, and subject info.
- When the root CA validates this, we get the leaf certificate which holds the signature above other things.
- If you want to use your own intermediate CA (for multi-cluster), you can specify it as a secret in the `istio-system` namespace named `cacerts` and then mounting that secret to `istiod`.
- The secret should contain the root cert, cert chain, ca cert and ca key.
- The aforementioned approach isn't fully advised since the secret is exposed as a secret in the namespace. Alternatively, you can "only loading the intermediate CA into memory".

