g(anything)**R**emote**P**rocedural**C**alls
Framework to make **remote** **calls** to make **procedurals**
> Some what like [[API]], but different in architecture (relying on HTTP methods) and different purpose ([[API]] is resource focusing)

gRPC is focusing on ***calling procedurals*** without relying on HTTP methods

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


gRPC uses [[HTTP2.0]] 

Also headers in gRPC are handled differently, in requests can be send only those headers that is different from the headers from the previous request

All of this gives much better preformance