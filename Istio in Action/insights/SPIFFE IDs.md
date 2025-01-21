- SPIFFE IDs are also called workload identities.
- An SVID (SPIFFE Verifiable Identity Document) is a verifiable workload identity.
- Istio uses X.509 certs for SVIDs. It encodes the SPIFFE ID as as URI in the Subject Alternative Name (SAN) extensions. This allows them to also use it for mTLS.
- Kube API sends a JWT token payload to every new pod to securely talk to it. It contains things like the secret name, namespace, service account name, etc.
- The Pilot agent that exists on Envoy decodes this JWT token and uses it to create the SPIFFE ID.

```
spiffe://cluster.local/ns/istioinaction/sa/default
```

- The Pilot agent uses the Secrets Discovery Service (SDS) to forward the cert and key to the Envoy proxy.
