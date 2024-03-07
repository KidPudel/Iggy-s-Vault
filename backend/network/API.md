> **Application Programming Interface** (API) server is the interface between the client and a database.
> API is running and listening to the calls from many clients. Meaning it is not one client per instance, it hosts all users, so the process, life of the program, is managed accordingly.

# Basic components
[info from](https://apidog.com/blog/what-are-the-components-of-an-api/#:~:text=APIs%20typically%20consist%20of%20three,data%20provided%20by%20the%20server.)


## API endpoints
endpoints are URL paths to navigate to find a specific request handler. Endpoints are refer to requests that can be made

- /users - Retrieve a list of users
- /users/123 - Get details of user with ID 123
- /posts - Create a new blog post

The endpoints define a structure of API

## Request handlers

Request handlers are functions that gets executed when hitting endpoint.

## HTTP request methods
HTTP request methods are there to indicate what action to perform on endpoint.
- GET: get/retrieve the resource
- POST: create new resource
- PUT: update the existing resource
- DELETE: delete resource 
 	> NOTE: we use post when adding body, because of the specifications (though get could have the body), GET should not have any body, since if the request method does not include defined semantics for an entity-body, then the message-body **SHOULD** be ignored when handling the request

## Request parameters
Parameters is the way to filter API requests and modify the flow of the request handler, as well as just provide an information add to Database for example

- Query string: Appended at the end of the URL `/users?status=active`
- Path: inserted into an endpoint path `/users/{id}`
- Header:  send a custom header like `Authorization`
- Body: Send in the request body, typically for POST/PUT methods

Parameters enable pagination, sorting, filtering, and more.

## Request headers
Headers make it easy to pass [[metadata]] to the endpoint without having to put it in the URL or the body.

In addition to standard `Content-Type`, we can define custom headers that provide additional context or functionality

Used for:
- Authentication: sending token or credentials
- Providing API version information
- Indicating the response type needed
- adding rate-limit context like tokens


## Request body
For POST and PUT, that create/update resources, *the body contains data itself*.  Data can be formatted into JSON, XML or others.
Body allows us to send complete resources or datasets, instead of individual parameters, for example sending all details about the order.

## api client
Api client refers to any application or system calling the API, meaning client, who initiates requests to the API.

## api response
Response is the answer from the api to the client, provides the requested data or confirmation of the result.
It will contain:
- status code
- response headers
- response data


## status codes
- 200 OK: request succeeded
- 201 Created: Resource successfully created
- 400 Bad request: Malformed request
- 404 Not found: Requested resource (endpoint), does not exist
- 500 Server Error: Internal server error

## Authentication
[[Authentication-authorization]] is required for public APIs to identify developers and restrict access. 
Common methods:
- API Tokens: Unique tokens assigned to each developer
- OAuth: Token-based authentication where apps get limited access
- Basic Auth: Sending username and password with the request

## Documentation
Documentation is the crucial part for developers to understand the API.
Documentation should provide:
- Overview of API's capabilities
-  Details on Authentication methods
- Information on endpoints, headers, body
- Example requests and resources
- Guidance on error handling
- Changelog

