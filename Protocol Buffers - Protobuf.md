In contrast to [[API]], where data is passed in text in JSON format with key value pairs
Protobuf is *Binary format*, meaning we send data in binary (010011001101), which makes it compressed data which  leads to smaller sizes of data to process therefore higher performance of data passing, but now we have to encode and decode data (serialization and deserialization).

Protobuf has strong typing with predefined data and data types that we want to send/retrieve and rules for them.
So we don't need any validation


# Defining a data model
```idl
message Person {
	; where value is for encoding/decoding of messages
	required string name = 1;
	required int32 age = 2;
	optional int32 salary = 3;
	; arrays
	repeated string hobbies = 4;
	; dict and injecting
	optional map <string, Person> children = 5;
}
```

After that we run compiler that will generate all functions we need to create, encode and decode data
> We can use proto files without gRPC, for example for data storage


# Create a gRPC Service itself
Here we can define what methods we can call and what models it takes and what returns
```idl
service SearchService {
	rpc Search(PersonRequest) returns (PersonResponse) {}
	rpc Notify(PersonRequest) returns (google.protobuf.Empty) {}
}
```

- **On the server side** where handling of methods is happening we define gRPC Server and opening a port for server to listen to catch request and handle them (decode, process, encode, and pass result back)
- **On the client side** where requests are made, those methods are called we define gRPC Stub, this service is for creating messages and sending them to the server
> So when making request, we go to the gRPC Stub, it creates message, encodes it, sends it to the gRPC server, server catches it, handles, and passes the response back in the reverse order.

> NOTE: On the handlers, we can also define a custom set of logic, to adjust *for* a request

> Also we can generate code for creating services in different languages, just compile them later


gRPC uses [[HTTP2.0]], which uses binary format
