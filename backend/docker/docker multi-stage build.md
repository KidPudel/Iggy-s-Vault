Multi stage build if to optimize the Dockerfile to easy to read and maintain, and which can lead for smaller docker image with no redundant data, selectively copying artifacts from previous stage.
Example we need to build some go project, we install dependencies and build, but what we actually need is only a binary build, so in the next stage with mach smaller base image, we just use that binary *from* previous build stage


```Dockerfile
# build stage
FROM golang:1.21
WORKDIR /src

RUN go install pack
COPY ./main.go 
RUN go build -o /bin/hello ./main.go

# prod stage
FROM scratch
# here we reference first stage by pointing to 0th `FROM`
COPY --from=0 /bin/hello /bin/hello
CMD ["/bin/hello"]
```

# name stage
We also can name stages
```Dockerfile
FROM node:21.6.1-alpine AS builder

...

# and reference
COPY --from=builder /go/src/github.com/name/proj/app .
```