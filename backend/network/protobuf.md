# Protocol Buffers (Protobuf)

A language-neutral binary serialization format developed by Google that encodes structured data more compactly than JSON.

## What it does

Data schemas are defined in `.proto` files with typed fields and field numbers. The `protoc` compiler generates code for serialization and deserialization in the target language.

Encoded data is binary — field names are not transmitted, only field numbers. This makes messages compact but not human-readable.

Deserialization requires the `.proto` schema. Unknown fields (from a newer schema version) are preserved but ignored by older code, enabling forward/backward compatibility.

## Code

```proto
syntax = "proto3";

message User {
  int32 id = 1;
  string name = 2;
}
```

```go
user := &pb.User{Id: 1, Name: "Alice"}
data, _ := proto.Marshal(user)
```

## Sources

- https://protobuf.dev/

## Related

- [[gRPC]]
- [[JSON]]
- [[serialization]]
- [[API]]

## Process

- What happens when a receiver encounters a field number it doesn't recognize?
- How does the binary format affect debugging compared to JSON?
- What does strong typing in Protobuf eliminate that you'd need to do manually with JSON?
