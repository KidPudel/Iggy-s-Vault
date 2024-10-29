UDP/IP - User Datagram Protocol

UDP unlike [[TCP]] is a [[connectionless protocol]], it speeds up communications by **not formally establishing a connection before data is transferred**,
but this results in a risk of [[packets]] (in UDP, called "*datagrams*") to become lost in transit, and also creates an opportunity for  exploitation in the form of [[DDoS attacks]]


UDP does the same as [[TCP]], but it doesn't verify its integrity, which makes it faster (perfect for games)


To solve this problem with order, [[buffer]] is commonly used
For example you can see it in youtube video, where there is a grey line that indicates, the data has been received, but not has been displayed on the screen, and when client replaying that information from the buffer, we can correct the order from the buffer.
