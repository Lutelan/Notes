The most basic step to start testing an API for issues, is to somehow get hold of some kind of documentation. Either human readable or computer readable both work. Most API documentation is openly available, if it is not, one may be able to access it by browsing applications which use the API.

Basically looking for endpoints which might hint to API documentation such as `/api`, `/swagger/index.html`, `openapi.json`. All this can also be used to identify various API endpoints. Once one has a list of API endpoints, various HTTP methods can be used on them the most common ones including `GET`, `PATCH` which adds data and `OPTIONS` which gives out information regarding which HTTP methods are supported by the endpoint.

Changing content types for endpoints can also help find errors which reveal information. Endpoints can also be found using wordlists of common industry terms used for endpoints.

Besides finding endpoints, finding parameters can also be useful since they can unlock hidden functionality. These parameters can be found using guessing using tools such as ParamMiner or just smart thinking or using `GET` requests on endpoints to find other parameters. These are especially useful for mass assignment vulnerabilities which allow one to overwrite parameters on the server side, since it helps find parameters to exploit.

##### Server Side Parameter Pollution in APIs
We can introduce new parameters or overwrite existing ones if their is a way to perform injection of characters, consider for example if a application uses a user's input in a query string to make requests on the server side to APIs. Using characters(URL encoded possibly) such as `#`, `&` and `=` and looking at the outputs. 

`#` can truncate query strings and cause parameters in server side requests to go unused, for example
```
GET /userSearch?name=peter%23foo&back=/home --> (server side request) GET /users/search?name=peter#foo&publicProfile=true
```
Similarly one can inject invalid parameters using `&` or even valid parameters to overwrite existing depending on the framework being used. 

Server side parameter pollution can also be included with REST paths, integrating path traversal issues. Say a website allows one to edit profiles using a request like `GET /edit_profile?name=peter` which results in a server side request which looks like `GET /api/private/user/peter`, if one submits path traversal sequences such as `/../` maybe one could edit other users data. Resulting in requests like `GET /api/private/users/peter/../admin`.

Server side pollution can also be performed in structured data formats such as JSON or XML, injecting parameters or modifying them just as before.