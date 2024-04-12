HTTP request methods are there to indicate what action to perform on endpoint.
- GET: get/retrieve the resource
- POST: create new resource
- PUT: update the existing resource
- DELETE: delete resource 
 	> NOTE: we use post when adding body, because of the specifications (though get could have the body), GET should not have any body, since if the request method does not include defined semantics for an entity-body, then the message-body **SHOULD** be ignored when handling the request


# more info
1. **GET**: This method is used to retrieve data from the server. When you enter a URL in your web browser or click on a link, your browser sends a GET request to the server to fetch the requested resource.
2. **POST**: This method is used to submit data to be processed by the server. It is commonly used in web forms, where the form data is sent to the server for processing. For example, when you submit a registration form or make an online purchase, a POST request is sent to the server with the form data.
3. **PUT**: This method is used to update or replace an existing resource on the server with the data provided in the request body. It is often used to modify or overwrite a complete resource.
4. **PATCH**: Similar to PUT, but PATCH is used to update or modify a partial resource on the server. It is more efficient than PUT when you only need to update a specific part of a resource.
5. **DELETE**: As the name suggests, this method is used to delete a resource from the server. For example, when you delete a file or remove an item from an online shopping cart, a DELETE request is sent to the server.
6. **HEAD**: This method is similar to GET, but it retrieves only the response headers from the server, without the response body. It is useful for checking the metadata of a resource or testing the resource's availability.
7. **OPTIONS**: This method is used to retrieve the communication options available for a particular resource or server. It is useful for determining the allowed methods, headers, and other options supported by the server.
8. **TRACE**: This method is used for debugging purposes. It echoes the received request back to the client, allowing the client to see what data is being received at the server.
9. **CONNECT**: This method is used to establish a tunnel to the server for protocols other than HTTP, such as SSL/TLS.