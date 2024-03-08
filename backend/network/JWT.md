JSON Web Token
This is a secure way of communicating, passing information between two parties, because this information is digitally signed

# Usage
- **Authorization**: This is the most common scenario for using JWT. Once the user is logged in, each subsequent request will include the JWT, allowing the user to access routes, services, and resources that are permitted with that token. Single Sign On is a feature that widely uses JWT nowadays, because of its small overhead and its ability to be easily used across different domains.
    
- **Information Exchange**: JSON Web Tokens are a good way of securely transmitting information between parties. Because JWTs can be signed—for example, using public/private key pairs—you can be sure the senders are who they say they are. Additionally, as the signature is calculated using the **header** and the **payload**, you can also verify that the content hasn't been tampered with.

# Structure
JWT consists of 3 parts separated by dots:
- Header
- Payload
- Signature

`xxxxx.yyyyyy.zzzzz`

## Header
Header consists of 2 parts, the algorithm being used (SHA256) and type of the token, which is JWT, that is then encoded in [[base64-encoding]]
```json
{ 
	"alg": "HS256",
	"typ": "JWT" 
}
```

## Payload
[[Payload]] contains the [[claims]]. that is encoded in [[base64-encoding]]

## Signature
To create the signature part you have to take the encoded header, the encoded payload, a secret, the algorithm specified in the header, and sign that.
For example if you want to use the HMAC SHA256 algorithm, the signature will be created in the following way:

```
HMACSHA256(
  base64UrlEncode(header) + "." +
  base64UrlEncode(payload),
  secret)
```

---

# Flow
1. user sends it's credentials
2. server verifies and generates a JWT token based on that, as well as private key
3. server stores private key associates with JWT as a key, in a *minimal* table
4. sends back JWT as a response to the user in secured [[cookies]] or/and params with HTTPS ([[internet-protocol]])
5. user stores JWT token in cookies or persistent storage
6. user sends JWT token in the subsequent requests
7. server matches JWT to the private key, and then decipher it and gets info like, user id, to then retrieve user unique data.

# Security
- We need to store the JWT on the client computer. If we store it in a LocalStorage/SessionStorage then it can be easily grabbed by an [[XSS]] attack
- If we store it in cookies then a hacker can use it (without reading it) in a [[CSRF]] attack and impersonate the user and contact our API and send requests to do actions or get information on behalf of a user.
But for that we have [[cookies]] flags
-  **httpOnly**, which makes it impossible to read cookie
- **SameSite=strict** cookie flag to prevent CSRF attack
- **secure=true,** to ensure that we can set cookies only in encrypted connection

and for mobile we use `PrivateMode` for security

# JWT vs Sessions
JWT in contrast to [[Sessions]]:
- JWT can store additional information, to performa server-side logic, like notification of expiration
- Sessions cannot do that, which forces table (storage of sessions) to has more data in it

JWTs are ideal for stateless, distributed systems with a focus on scalability and single sign-on, while **session-based approaches are more appropriate for applications that prioritise server-side control, robust session management, and sensitive data protection**.

Sessions 
1. server gets session id
2. makes a lookup **request** for data by session id
3. uses data to perform authenticated **request**

JWT
1. server retrieves JWT
2. makes a lookup **request** for private token
3. decipher data from JWT
4. uses data to performa authenticated **request**