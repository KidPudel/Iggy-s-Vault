# public/private keys (hashing)

the main point for that is to secure/encrypt the data

we can have a key that just the two of us will know

hello 🔒 → пwщуr → 🔓 hello

## but how to share the keys?

the solution is to have two keys that will encrypt/decrypt one another

if I’ll encrypt with public key, then nobody would decrypt it, because only private key could decrypt it but if encrypt with private key, you could decrypt it with public key, and the point it to prove that it’s me who encrypted the message

first key is public, you just share it everywhere or just with your friend, we both has **this** public key, and since we both have this key we can encrypt **it for me** → **because i can decrypt** it with my private key that is in this pair of keys

so it means that if it’s my keys, only i can decrypt it because we should not share private key over the internet or else where cus it could be stolen → if we want to encrypt messages for each other and decrypt it, we must have two pairs of keys (me [🔑, 🔐] and friend [🔑, 🔐])

> _**We ask someone to encrypt something for us and only we can decrypt it**_

# Digital signature

## How it works?

Basically _**digital signature**_ is just a ~~_waring, terms_~~ _**hash value with embedded private key in it**_, that’s ALL!!

now to straight to the flow of it:

1. key generation, signer generates private and public keys
2. writes a message
3. make it fixed-sized hash value with hash function (Message Digest Algoritms like SHA-2 , SHA-256 (Secure Hash Algorithm))
    1. additionally we could use Cipher algorithms to secure transition of the data
4. embedding a private key with a signature algorithm SHA-2 family with ECDSA (Elliptic Curve Digital Signature Algorithms) (such as `SHA256withECDSA`)
    1. additionally could be used MAC (Message Authentication Code) algorithm for additional security with a synchronous cryptography
5. after that the digital signature is attached to the document
6. recipient gets a document (via email or something else)
7. calculating the document with the same hash function independently
8. decrypting the signature with a public key via decryption algorithm
9. comparing hash values
10. identity verification with public key (public key gets accepted means private key is the one → therefore identity is checked)

🔑1🔐1(decrypt message encrypted from our public key)🔑2(decrypt message encrypted by friend’s private key

🔑2🔐2🔑1

1. Key Generation: The signer generates a key pair consisting of a private key and a corresponding public key. The private key is securely stored by the signer, while the public key is made available to others.
    
2. Document Preparation: The signer prepares the document that needs to be digitally signed. This could be a contract, a legal agreement, an electronic form, or any other digital document.
    
3. Hashing: The signer calculates the hash value of the document using a cryptographic hash function. The hash function converts the document into a fixed-length string of characters unique to that document.
    
4. Signing: The signer applies their private key to the hash value using a signing algorithm. This process creates the digital signature, which is essentially an encrypted version of the hash value.
    
5. Signature Attachment: The digital signature is attached to the document. This can be done by embedding the signature within the document itself or by creating a separate file that includes the signature.
    
6. Distribution: The digitally signed document is distributed to the intended recipients or made available for download. This can be done through email, file sharing platforms, secure document repositories, or any other appropriate means of distribution.
    
7. Signature Verification: Recipients of the digitally signed document perform the following steps to verify the signature:
    
    a. Obtain the Signer's Public Key: The recipients obtain the signer's public key from a trusted source. This could be from the signer directly, a trusted certificate authority, or a public key infrastructure.
    
    b. Hash Calculation: The recipients independently calculate the hash value of the received document using the same hash function as the signer.
    
    c. Signature Decryption: The recipients use the signer's public key and a verification algorithm to decrypt the digital signature. This produces a decrypted hash value.
    
    d. Hash Comparison: The recipients compare the decrypted hash value with the calculated hash value of the received document. If the two values match, it confirms that the document has not been altered since the signature was created, and the signature is valid.
    
    e. Identity Verification: Optionally, the recipients can verify the identity of the signer using the public key. This can involve cross-referencing the public key with a trusted source or certificate authority.
    
8. Result: If the signature verification is successful, the recipients can trust that the document is authentic, unaltered, and originated from the claimed signer. They can proceed with confidence in the integrity and validity of the digitally signed document.
    

The flow described above outlines the general process of using digital signatures. However, specific implementations and tools may have additional steps or variations based on the underlying technologies and security protocols employed.

Certainly! In the "e. Identity Verification" step, recipients can verify the identity of the signer by using the public key associated with the digital signature. Here's a more in-depth explanation of how this verification process can occur:

1. Public Key Retrieval: The recipients obtain the signer's public key from a trusted source. This can be done in various ways, depending on the digital signature infrastructure in place. Common methods include:
    - Direct Sharing: The signer may provide their public key directly to the recipients, either in person, through email, or via a secure messaging system.
    - Certificate Authorities (CAs): CAs are trusted entities that issue digital certificates. A digital certificate contains the signer's public key along with additional information, such as their name, organization, and a digital signature from the CA. Recipients can obtain the signer's public key by validating the digital certificate from a trusted CA.
    - Public Key Infrastructure (PKI): PKI is a framework that provides a hierarchical system of CAs, enabling a network of trust. Recipients can retrieve the signer's public key from a trusted CA within the PKI.
2. Trust and Verification: Once the public key is obtained, recipients can proceed with the identity verification process. Here are a few common approaches:
    - Trusted Sources: Recipients may have a pre-established list of trusted sources (individuals, organizations, or CAs) from which they accept public keys. They cross-reference the received public key with this trusted list to verify its authenticity and trustworthiness.
    - Certificate Chain Verification: If a digital certificate was used to distribute the public key, the recipients can verify the authenticity of the certificate by validating the digital signature of the issuing CA. This involves checking the CA's digital signature against the trusted CA's public key. The verification process may continue recursively up the certificate chain until a trusted CA is reached.
    - Web of Trust: In certain scenarios, a decentralized approach called a "web of trust" can be used. This involves establishing trust based on recommendations and introductions from individuals who are already trusted. Recipients rely on endorsements from known and trusted parties to verify the signer's identity.
    - Manual Verification: In some cases, recipients may need to manually verify the identity of the signer. This can involve contacting the signer directly, conducting background checks, or using other means to ensure the authenticity of the public key.

By employing one or a combination of these methods, recipients can gain confidence in the identity of the signer associated with the public key. This verification step is crucial in ensuring that the digital signature is not only valid but also originates from the claimed source, providing an additional layer of trust and authenticity to the digitally signed document.

# PGP

In the context of PGP (Pretty Good Privacy), there is no such thing as a "PGP certificate." However, PGP does use certificates as part of its key management system.

PGP is a cryptographic software program used for encryption and digital signatures. It uses public-key cryptography, where each user has a pair of keys: a public key and a private key. The public key is shared with others, while the private key is kept secure and known only to the owner.

In PGP, certificates are used to bind a user's identity to their public key. These certificates are known as "PGP certificates" or "PGP keys" in common usage, although they are not in the traditional X.509 certificate format.

A PGP certificate typically includes the following information:

1. User Identity: The user's name and email address.
2. Public Key: The user's public key, which is used for encryption and verifying digital signatures.
3. Certification Information: Information about the certification process, such as the name of the certifying authority and the signature of the certifier.

The purpose of a PGP certificate is to establish trust in the association between the user's identity and their public key. When someone wants to communicate securely with a PGP user, they can verify the user's identity by checking their PGP certificate. This verification involves checking the signature on the certificate against the known public key of the certifying authority.

PGP certificates are typically distributed through key servers or exchanged directly between users. They are used to establish a web of trust, where users can vouch for the authenticity of each other's public keys by signing each other's certificates.

It's important to note that the term "PGP certificate" is not a standardized term, and PGP uses its own key management system distinct from traditional X.509 certificates used in other cryptographic protocols like SSL/TLS.