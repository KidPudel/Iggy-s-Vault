g(anything)**R**emote**P**rocedural**C**alls
Framework to make **remote** **calls** to make **procedurals**

In [[RESTful API]] we rely on [[URL]], in gRPC this mechanism of hitting the destination is fundamentally different. It relies on defining an *isolated* contract by ***service definition***

gRPC is focusing on ***calling procedurals*** without relying on HTTP methods, so it is a perfect fit for communicating between different **internal services** in micro service architecture 

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

# Create a gRPC Service itself
---
We define service which is the collection of methods to call with models it takes in request and returns in response
We need to define this service only once, like a 'prototype'/protocol/contract and compile it to client and server
```idl
option go_package="hello-grpc/lostpetsearch";

package lostpetsearch;


; best if name is domain bound
service SearchService {
	; methods
	rpc Search(PersonRequest) returns (PersonResponse) {}
	rpc Notify(PersonRequest) returns (google.protobuf.Empty) {}
}
```

- **On the server side** where handling of methods is happening we define gRPC Server and opening a port for server to listen to catch request and handle them (decode, process, encode, and pass result back)
- **On the client side** where requests are made, those methods are called we define gRPC Stub, this service is for creating messages and sending them to the server
> So when making request, client go to the gRPC Stub *Service*, it creates message, encodes it, sends it to the gRPC server, server catches it, handles, and passes the response back in the reverse order.


# Defining a data model
---

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

# Defining methods
---
There is 4 types of methods corresponding to 4 types of gRPC ways of handling request [[gRPC Streams]]
## Unary RPC
```idl
rpc GetPerson(PersonID) returns (Person) {}
```

## Server-Side Streaming
```idl
rpc TrackPerson(PersonID) returns (stream PersonPosition) {}
```

## Client-Side Streaming
```idl
rpc UploadChunk(stream PhotoChunk) returns (UploadResultStatus) {}
```

## Bidirectional Streaming
```idl
rpc FindEachOther(stream DeliveryPosition) returns (stream PersonPosition) {}
```

# Generating
---

After that we run compiler that will generate all functions we need to create, encode and decode data
> We can use proto files without gRPC, for example for data storage


```zsh
protoc --go_out=. --go_opt=paths=source_relative \
    --go-grpc_out=. --go-grpc_opt=paths=source_relative \
    routeguide/route_guide.proto


protoc --go_out=. --go_opt=paths=source_relative ./redis/models/proto/models.proto
protoc --go_out=. --go_opt=paths=source_relative ./kafka/models/proto/orders/order.proto
protoc --go_out=. --go_opt=paths=source_relative ./kafka/models/proto/offers/offer.proto
protoc --go_out=. --go_opt=paths=source_relative ./kafka/models/proto/notifications/notifications.proto

```

this will generate
- `.pb.go` structs corresponding to protobuf and helper methods to populate, serialize, retrieve request and response message types
- `_grpc.pb.go` contains server and client specific code
	- an interface type for servers to implement
	- interface type for stub clients to call with the methods defined
	- registration functions with a gRPC server


# Implementing generated skeleton
---
## Server
---
First prepare the base by generating implementations
```go
package server

import (
	"context"
	"google.golang.org/grpc"
	pb "hello-grpc/lostpetsearch"
)

type PetSearchServer struct {
	pb.LostPetSearchServiceServer
}

func (petsearchserver *PetSearchServer) GetPetNearby(_ context.Context, _ *pb.Position) (*pb.PetID, error) {
	panic("not implemented") // TODO: Implement
}

func (petsearchserver *PetSearchServer) TrackPet(_ *pb.PetID, _ grpc.ServerStreamingServer[pb.Position]) error {
	panic("not implemented") // TODO: Implement
}

func (petsearchserver *PetSearchServer) UploadPicture(_ grpc.ClientStreamingServer[lostpetsearch.PhotoChunk, lostpetsearch.Status]) error {
	panic("not implemented") // TODO: Implement
}

func (petsearchserver *PetSearchServer) FindEachOther(_ grpc.BidiStreamingServer[lostpetsearch.Position, lostpetsearch.Position]) error {
	panic("not implemented") // TODO: Implement
}

```

Now lets implement those methods

### Unary
With single RPC request it is basic, just get value, do some work and return value from DB
request -> response. done
```go
// just an example
func (petsearchserver *PetSearchServer) GetPetNearby(ctx context.Context, position *pb.Position) (*pb.PetID, error) {
	fmt.Println("position is: ", position)
	return &pb.PetID{
		Id: position.Lat,
	}, nil
}
// something common to gRPC can be found by "how to _?"
```

### SSS
Here we stream to the client until we done
We send data via methods like `Send`
```go
func (petsearchserver *PetSearchServer) TrackPet(petID *pb.PetID, stream grpc.ServerStreamingServer[pb.Position]) error {
	fmt.Println("petID is: ", petID)
	for i := 0; i < 10; i++ {
		if err := stream.Send(&pb.Position{
			Lat:  proto.Int32(int32(i * 2)),
			Long: proto.Int32(int32(i * 3)),
		}); err != nil {
			return err
		}
	}
	return nil
}
```


### CSS
Here we accept the stream from client, and once we read all, we send back to the stream the response and close the stream
```go
func (petsearchserver *PetSearchServer) UploadPicture(stream grpc.ClientStreamingServer[pb.PhotoChunk, pb.Status]) error {
	photo := make([]*pb.PhotoChunk, 0)
	for {
		photoChunk, err := stream.Recv()
		if err == io.EOF {
			// read whole client stream, time to response
			if err := stream.SendAndClose(&pb.Status{
				Success: proto.Bool(true),
			}); err != nil {
				return err
			}
			fmt.Println(photo)
			return nil
		}
		photo = append(photo, photoChunk)
	}
}
```

### Bidirectional streaming
This is a combination of SSS and CSS
```go
func (petsearchserver *PetSearchServer) FindEachOther(stream grpc.BidiStreamingServer[pb.Position, pb.Position]) error {
	for {
		clientPosition, err := stream.Recv()
		if err == io.EOF {
			return nil
		}
		if err != nil {
			return err
		}

		fmt.Println("got from client: ", clientPosition)

		latestPositios := []*pb.Position{
			{
				Lat:  proto.Int32(rand.Int31()),
				Long: proto.Int32(rand.Int31()),
			},
			{
				Lat:  proto.Int32(rand.Int31()),
				Long: proto.Int32(rand.Int31()),
			},
			{
				Lat:  proto.Int32(rand.Int31()),
				Long: proto.Int32(rand.Int31()),
			},
		}
		for _, latestPosition := range latestPositios {
			if err = stream.Send(latestPosition); err != nil {
				return err
			}
		}
	}
}
```

### Start a server
```go
func main() {
	listenConfig, err := net.Listen("tcp", "localhost:8000")
	if err != nil {
		log.Fatal(err)
		return
	}
	grpcServer := grpc.NewServer()
	pb.RegisterLostPetSearchServiceServer(grpcServer, server.NewServer())
	grpcServer.Serve(listenConfig)
}
```

## Client
---
In order for client side to communicate with server it needs to create a gRPC channel
This is done via `grpc.NewClient`
```go
	serverAddr := flag.String("addr", "localhost:50051", "The server address in the format of host:port")
	conn, err := grpc.NewClient(*serverAddr)
	if err != nil {
		log.Fatal(err)
	}
	defer conn.Close()

```
now to connect with our implementation of the client side
```go
client := pb.NewLostPetSearchServiceClient(conn)
```

### Unary
```go
	position := &pb.Position{
		Lat:  proto.Int32(rand.Int31()),
		Long: proto.Int32(rand.Int31()),
	}
	petID, err := client.GetPetNearby(ctx, position)

```
### SSS
```go
	stream, err := client.TrackPet(ctx, petID)
	for {
		position, err := stream.Recv()
		if err == io.EOF {
			fmt.Println("done")
			break
		}
		fmt.Println(position)
	}

```
### CSS
```go
	clientStream, err := client.UploadPicture(ctx)
	for i := 0; i < 10; i++ {
		if err = clientStream.Send(&pb.PhotoChunk{
			Byte: proto.Int32(rand.Int31()),
		}); err != nil {
			log.Fatal(err)
		}
	}

```
### Bidirectional Streaming
```go
	biStream, err := client.FindEachOther(ctx)
	waitCh := make(chan int)
	go func() {
		for {
			servPos, err := biStream.Recv()
			if err == io.EOF {
				close(waitCh)
			}
			fmt.Println(servPos)
		}
	}()

	for i := 0; i < 10; i++ {
		if err = biStream.Send(position); err != nil {
			log.Fatal(err)
		}
	}
	biStream.CloseSend()
	<-waitCh

```



gRPC uses [[HTTP2.0]], which uses binary format

> Also *headers* in gRPC are *handled differently*, in requests can be send only those headers that is different from the headers from the previous request

All of this gives much better performance

> gRPC is *guarantee to maintain the order of the requests* and allows to cancel if one of the sides are got stuck a.k.a not responding.
It's a perfect match for Micro services.


But also gRPC can be used outside of microservices, we can use it in web app, utilizing gRPC-Web which allows making regular single request and getting stream from the server 



[[gRPC Streams]]


It's not that deep, all those technologies, are tools, and tools are for different usecases, it's natural.



# oneOF
when only one field from a set is possible fields. This saves memory 
```proto
syntax = "proto3";

message Payment {
  string id = 1;
  
  oneof method {
    string credit_card = 2;
    string paypal = 3;
    string bank_transfer = 4;
  }
}

```