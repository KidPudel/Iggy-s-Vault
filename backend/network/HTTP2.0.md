Hyper Text Transfer Protocol 2.0 uses binary format, so it compressed well
![[Pasted image 20240613224721.png]]

Also in HTTP2.0 there is [[multiplexing]], which allows to remain connection open to reduce the number of [[TCP]] connections, instead of creating it each request like in HTTP1, which allows for passing stream of requests in different ways: [[gRPC Streams]]

![[Pasted image 20240613224948.png]]