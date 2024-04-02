HTTP request methods are there to indicate what action to perform on endpoint.
- GET: get/retrieve the resource
- POST: create new resource
- PUT: update the existing resource
- DELETE: delete resource 
 	> NOTE: we use post when adding body, because of the specifications (though get could have the body), GET should not have any body, since if the request method does not include defined semantics for an entity-body, then the message-body **SHOULD** be ignored when handling the request
