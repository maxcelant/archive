- You can configure "SNI passthrough" on your east-west gateways. What that means is that the gateway won't decrypt the traffic request and re-route it to the destination. It will just use the SNI packet header to figure out where it should go. 
	- `CLUSTER: outbound_.80_._.catalog.default.svc.cluster.local`
- By default, istiod acts as a root CA by storing the root cert and key in-memory. It then signs leaf certs using 
- As a reminder, A Certificate Signing Request (CSR) contains:

```json
{
  "subject": {
    "commonName": "example.com",
    "organization": "Example Inc.",
    "organizationalUnit": "IT Department",
    "locality": "San Francisco",
    "state": "California",
    "country": "US"
  },
  "publicKey": "-----BEGIN PUBLIC KEY-----...-----END PUBLIC KEY-----",
  "metadata": {
    "csrVersion": 1,
    "signatureAlgorithm": "SHA256withRSA"
  }
}
```

- When the root CA validates this, we get the leaf certificate.

```json
{
  "certificate": {
    "subject": {
      "commonName": "example.com",
      "organization": "Example Inc.",
      "organizationalUnit": "IT Department",
      "locality": "San Francisco",
      "state": "California",
      "country": "US"
    },
    "publicKey": "-----BEGIN PUBLIC KEY-----...-----END PUBLIC KEY-----",
    "issuer": {
      "commonName": "Example CA",
      "organization": "Example Certificate Authority",
      "country": "US"
    },
    "validity": {
      "notBefore": "2025-01-01T00:00:00Z",
      "notAfter": "2026-01-01T00:00:00Z"
    },
    "serialNumber": "1234567890ABCDEF",
    "signatureAlgorithm": "SHA256withRSA",
    "signature": "-----BEGIN SIGNATURE-----...-----END SIGNATURE-----"
  }
}
```


