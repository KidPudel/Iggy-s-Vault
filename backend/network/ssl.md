# SSL/TLS

A cryptographic protocol that establishes an encrypted channel between a client and server over a network.

## What it does

TLS (Transport Layer Security) is the current name; SSL (Secure Sockets Layer) is the deprecated predecessor but the name persists.

The TLS handshake sequence:
1. Client sends `ClientHello` with supported cipher suites and TLS version
2. Server responds with `ServerHello`, chosen cipher suite, and its certificate
3. Client verifies the certificate against a trusted CA
4. Key exchange occurs (e.g. ECDHE) to produce a shared session key
5. Both sides confirm with a `Finished` message encrypted with the session key
6. Application data flows encrypted with symmetric encryption (e.g. AES-GCM)

A TLS certificate binds a public key to a domain name, signed by a Certificate Authority.

## Sources

- https://www.rfc-editor.org/rfc/rfc8446

## Related

- [[three-way-handshake]]
- [[HTTP]]
- [[TCP]]
- [[certificates]]
- [[hashing]]

## Process

- What happens if the CA that signed a server's certificate is compromised?
- How does TLS fit on top of TCP — which handshake runs first?
- What is the difference between symmetric and asymmetric encryption in the context of a TLS session?
- Where does certificate pinning change the normal TLS flow?
