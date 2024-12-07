In contrast to [[API]], where data is passed in text in JSON format with key value pairs
Protobuf is *Binary format*, meaning we send data in binary (010011001101), which makes it compressed data, which leads to smaller sizes of data to process, therefore higher performance of data passing, but now we have to encode and decode data (serialization and deserialization).

Protobuf has strong typing with predefined data and data types that we want to send/retrieve and rules for them.
So we don't need any validation

Protobufs allows to serialize structured data into bytes which is smaller and faster than [[JSON]]