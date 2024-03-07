HTTP headers is the key(case insensitive)-value parameters that lets clients and a server to pass additional information ([[metadata]]) along with HTTP requests or responses.
Therefore headers provide [[metadata]] about HTTP requests/responses. 
While the [[Payload]] (body) consist of the actual data being transferred, headers provide the context to the payload.

> NOTE: While spaces between an colon the value is ignored

# examples
- The `Content-Type` header specifies the type of data in the payload (JSON, XML, text, etc.)
- The `Content-Length` header indicates the size of the payload data in bytes
- The [[Authorization header]] provides metadata about the credentials used for authentication
- The `Cache-Control` header gives instructions on caching policies for the response
- The `Referer` (sic) header contains metadata about the previous URL the request came from