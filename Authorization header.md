Authorization header provides a metadata about credentials that used for authentication.

# types
1. **Basic Authentication**:
	- Used for sending [[base64-encoding]] credentials (username:password) in the header
	- simple, but insecure *over unencrypted* connections as it transmits credentials in plaintext
	- Format: `Authorization: Basic <credentials>`
2. **Bearer Authentication**:
	- Used for transmitting secure [[Authentication-authorization]] token like [[JWT]], or an OAuth access token, because of:
		- Separation of credentials, because of bearer token opaque nature, so tokens doesn't store credentials directly.
		- Signed and encrypted tokens, which ensures that only intended parties can read the data of token
	- Format: `Authorization: Bearer <credentials>`
	- Widely used in API and web services for token-based authentication and authorization
1. **Digest authentication**:
	- Used for sending digests ([[hashing]]) versions of username, password and other data
	- Format: `Authorization: Digest <credentials>`
2. **API Key Authentication**:
    - Used for transmitting pre-shared API keys for authentication.
    - Format: `Authorization: ApiKey <api_key>`
    - Commonly used for authenticating requests to public APIs or service-to-service communication.
3. **AWS Signature Authentication**:
    - Used for authenticating requests to Amazon Web Services (AWS) APIs.
    - Format: `Authorization: AWS4-HMAC-SHA256 <authentication-details>`
    - Includes access key, signature, and other AWS-specific authentication details.
4. **OAuth Authentication**:
    - Used for transmitting OAuth access tokens for authentication and authorization.
    - Format: `Authorization: Bearer <oauth_token>` (same as Bearer Authentication)
    - Widely adopted for delegated authentication and authorization scenarios.