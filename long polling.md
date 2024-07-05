Like [[short polling]], but more *patient* version
Instead of in a high frequency asking for updates and getting the response from the server right away, resulting in frequent checking, **server holds the request open for a while to *process***, resulting in a room for the server to process, resulting in reducing the update checking

But it's still not perfect.
- you are waiting on a line
- still can be tough on server resources
Here comes the [[webhooks]]