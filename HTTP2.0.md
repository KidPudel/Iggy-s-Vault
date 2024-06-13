Hyper Text Transfer Protocol 2.0 uses binary format, so it compressed well
![[Pasted image 20240613224721.png]]

Also in HTTP2.0 there is [[multiplexing]], which allows to remain connection open instead of creating it each request like in HTTP1, which allows for passing stream of requests in ways like [[Server, Client and Bidirectional streaming]]
gRPC is guarantee to maintain the order of the requests and allows to cancel if one of the sides are got stuck a.k.a not responding.
It's a perfect match for Micro services.

![[Pasted image 20240613224948.png]]