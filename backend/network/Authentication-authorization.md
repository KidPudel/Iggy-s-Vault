[info from](https://apidog.com/blog/api-authorization/)

**Authentication** is the process of exchanging credentials between the user and a browser, to verify the identity 

**Authorization** is the process of verifying the necessary permission for the *authenticated* user to access some information. Meaning it ensures that only authorised users can access the information


# Types of authorization


## API token Authorization
Simple method of generating unique keys for each API client.

### how
Clients must include this token on each request, to authenticate itself.

### use case
It is a good fit for application that don't require complex authentication and don't store sensitive user data


## OAuth 1.0 Authorization
OAuth 1.0 widely used authorization framework/protocol, that allows users to grant third-party application access to their resources (one website to access resources of another)

### how
OAuth 1.0 is based on concept of access token, which are issued to authorised clients and used to access protected resources

### use case
commonly used in social media APIs, where users want to share their data with other apps
1. **Request Token:** The client (the application that wants access) asks the server for a request token.
2. **Authorization:** The user is redirected to the server, where they log in and approve the request token.
3. **Access Token:** The approved request token is exchanged for an access token.
4. **Access Resources:** The access token is used to access the user's resources on the server.
### example
- logging to Reddit via Google account
- photo-sharing app (client), accesses Instragram's (server) photos, without giving it **your** login credentials 

### OAuth 2.0
- Token handling: Separates authorization and token issuance, allowing for more security and different token types
- Scopes: Define the specific permissions the client requires
- Support for mobile and Cloud
- Simplicity: More flexible, makes it easier for developers 
## JSON Web Token (JWT)
JWT is the popular method of securely transferring data between parties in JSON format.

