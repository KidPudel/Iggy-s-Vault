g(anything)**R**emote**P**rocedural**C**alls
Framework to make **remote** **calls** to make **procedurals**
> Some what like [[API]], but different in architecture (relying on HTTP methods) and different purpose ([[API]] is resource focusing)

gRPC is focusing on ***calling procedurals*** without relying on HTTP methods, so it is a perfect fit for communicating between different internal services in micro service architecture 

API example
```
PUT /products/123
{
	"price": "20.00"
}
```

jsonRPC (previous version of gRPC)
```
POST /producs
{
	"method": "update_price",
	"params": {
		"new_price": "20.00"
	}
}
```

Google got rid of JSON format like in [[RESTful API]]
They are utilizing [[Protocol Buffers - Protobuf]]

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
> So when making request, client go to the gRPC Stub *Service*, it creates message, encodes it, sends it to the gRPC server, server catches it, handles, and passes the response back in the reverse order.

> NOTE: On the handlers, we can also define a custom set of logic, to adjust *for* a request

> Also we can *generate code for creating services in different languages,* just compile them later


---

> gRPC uses [[HTTP2.0]], which uses binary format

> Also *headers* in gRPC are *handled differently*, in requests can be send only those headers that is different from the headers from the previous request

All of this gives much better performance

> gRPC is *guarantee to maintain the order of the requests* and allows to cancel if one of the sides are got stuck a.k.a not responding.
It's a perfect match for Micro services.


But also gRPC can be used outside of microservices, we can use it in web app, utilizing gRPC-Web which allows making regular single request and getting stream from the server 