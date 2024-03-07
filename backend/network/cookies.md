HTTP cookies are text files made up of tiny bits of data, stored in web browser, typically located in a header. 
Those bits are to track user's journey, enabling them to access features specific to the user.

They are sent between web server and browser to **remember the information about the user's session or preferences**.

> **NOTE:** from web browser (client), when making request to the server, stored *cookies are sent automatically*

Types of cookies:
1. **Session Cookies**: 
	1. Scoped to the browser life (until closed)
	2. Maintaining user login status
2. **Persistent Cookies:**
	1. Remain specified duration
	2. Remembering user preferences or login information
3. **First-party**:
	1. Issued by the website
	2. To enhance user experience
4. **Third-party**:
	1. Issued by other domain
	2. Used for tracking or advertising across the website
5. **Secure Cookies**:
	1. Sent only over encrypted connection
	2. Enhancing security by protecting sensitive information during transmission
6. **HttpOnly Cookies:**
    - Cannot be accessed through client-side scripts (JavaScript). Only server can read it
    - Helps prevent cross-site scripting (XSS) attacks.
7. **SameSite Cookies:**
    - Controls when cookies are sent with cross-site requests.
    - Can be set to "Strict" (cookies only sent in a first-party context), "Lax" (limited in cross-site requests), or "None" (no restrictions).