WebSockets is a communication protocol, that allows for establishing connection/session between two parts, to listen to requests and update data in real time via single [[TCP]] connection
This enables [[full duplex]] interaction between the client (like web browser or any other app) and the [[Web server]] with lower overhead than [[long polling]] or [[short polling]] that are half-duplex.


[[HTTP]] request: sends a request to the server → get response → drawing completed → connection closed.
but with WebSockets we can keep listen.

[[Webhooks]]