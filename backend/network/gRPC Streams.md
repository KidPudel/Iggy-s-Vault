# Unary RPC
Client sends singe request ->
Server sends singe response <-

# Server-Side Streaming
Client sends singe request ->
Server streams multiple responses back to the client <-<-<-
Use case: When server response with a large dataset [[batching]]

# Client-Side Streaming
Client streams a request ->->->
Server streams multiple responses back to the client <-
Use case: When client needs to send large request [[batching]] or multiple pieces of data in chunks 

# Bidirectional Streaming
Client streams a request ->->->
Server streams multiple responses back to the client <-<-<-
Use case: Best for real-time, interactive apps where both sides need to exchange data continuously and asynchronously. Basically like [[Web Sockets]]