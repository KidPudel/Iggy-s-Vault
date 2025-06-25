# synchronization
- `errgroup`: provides error handling mechanism for [[goroutines]]. Derived [[context]] is canceled the first time a function passed to Go returns error and Wait returns that error. Must have for golang services
- `workerpool`: manage pools of worker groups. [[concurrency in go]]
# utility
- `godotenv`: parse dotenv file
# architecture/flow
- `goeventbus`: go implementation of [[event bus]]
# logging
- `zap`: fast alternative to default `log`
- `zerolog`: very high performance JSON logger

# database
- `gorm`: [[orm]]

# HTTP/Web
2 types
- http handler function signature
- context with an error return, which allows you to clean handlers, and centralize error handling

- `gin` good performance
- `chi`: lightweight
- `echo`: high-performance


# Serialization
- `protobuf`: [[Protocol Buffers - Protobuf]] [[gRPC]]
- `easyjson`

# Errors
- `errors`: extended error handling with stack traces

# Testing
- `gomock`: [[mock]]ing service by Google that generates mock implementations from intefaces.o

# Services


# Distributed
- DTM framework for distributed transactions like [[saga]] [DTM](https://en.dtm.pub/)