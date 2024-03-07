> SSL - Security Socket Layer. Digital signature that verifies website’s identity, therefore encrypts connection. Security protocol (system of rules) generates encrypted link between web server and web browser

Socket - Communication endpoint that allows two computers to communicate with each other

# TSL

> TSL - Transfer Socket Protocol. A new version of SSL, but it’s still commonly called SSL

# Process

1. SSL handshake
    1. browser/server ties to connect to web server secured by SSL
    2. browser/server asks to identify web server
    3. web server sends copy of SSL certificate
    4. browser verifies identity by trusting SSL
        1. trusts, sends signal to web server
    5. web server sends digitally signed certificate to start SSL encrypted session
    6. data sharing now encrypted between two sides

you don’t need to do a mind process each time to understand what := is

it’s like to have a map

```go
thoughts := make(map[string]Definition)
thoughts["ssl"] = { def: "Secure Socket Protocol", subDef: {"Socket - communication endpoint between two sides", "Protocol - system of rules" } }
thoughts[":="] = { def: "declare a NEW variable with initial value" }
```

and instead of using those keys to quickly operate, we in the process ditch it and calculating definition again, again and again, not cool