# Important aspects to define

* using HTTP methods - (not only GET and POST but also PUT, PATCH, DELETE) - for implementing CRUD operations
* supported formats for requests and responses (possibly more than one format: JSON, XML, ecc)
* content negotiation
  * how client and server negotiate the format to use
  * `Accept: <format>` (client) <=> `Content-Type: <format>` (server)
* errors handling
  * significative error codes: `406 Not Acceptable`, `405 Method Not Allowed`, `400 Bad Request`, `422 Unprocessable Entity`, `204 No Content`, `501 Method Not Implemented`,  ecc (not just `500 Server Error`)
  * Standard formats, e.g. [Problem Detail](https://medium.com/some-tldrs/rfc-7807-problem-details-for-http-apis-is-published-28d77c4afba0) ([See rfc](https://tools.ietf.org/html/rfc7807))
    ```http
    HTTP/1.1 403 Forbidden
    Content-Type: application/problem+json
    Content-Language: en
    ```
    ```json
    {
        "type": "https://example.com/probs/out-of-credit", 
        "title": "You do not have enough credit.", 
        "detail": "Your current balance is 30, but that costs 50.",
        "instance": "/acc/12345/errors/out-of-credit/err_instance_id=123",
        "balance": 30,
        "account": "/acc/12345"
    }
    ```
* versioning:
  * version part of the URI, e.g. `/api/v1/contacts/1`
    * easy to implement and use
    * when version changes, all API calls must be rewritten
    * violates the uniqueness of a HTTP resource as should be by the REST-API principles
  * version inside the media type, e.g. `application/vnd.company.myapp.customer-v1.json`
    * version must be extracted from that string (functions to extract it easily should be provided)
    * URI stays the same when version changes
* filtering and validating input data
* authentication 
  * OAuth2, user/password, token, etc
* authorization
  * e.g. a user can only `GET` a resource, whereas an admin can `PUT`, `POST`, `PATCH`, `DELETE`, etc
  * RBAC - Role Based Access Control
  * ACL Access Control List
* documentation
  * e.g. Swagger

# Project

* define all possible client-server interactions
* define url addresses to be used

# REST - REpresentational State Transfer

A REST API is:

* stateless (depends only on HTTP requests content, state is maintained only at the client, not on the server)

* Level 0: HTTP protocol and data formats (e.g. JSON, XML, etc)
* Level 1: unique URI identifier for each resource (i.e. entity)
* Level 2: extended use of HTTP methods
* \[Level 3 (optional): relations as hypermedia links \] (optional level: when present the API is "RESTful" (i.e. full REST))
  * HATEOS - Hypermedia As The Engine Of (application) State

URLs
----

* `/resource-name[/resource-id]`
  * e.g. `/speakers[/speaker-id]` - the resource name is always  plural cause without an id it returns a collection of resources

Implementing CRUD via HTTP methods
----------------------------------

```http
POST /speakers HTTP/1.1
Accept: application/json
Content-Type: application/json
```
```json
{
    "name": "Enrico Zimuel"
}
```

```http
HTTP/1.1 201 Created
Content-Type: application/json
Location: /speakers/ezimuel
```
```json
{
    "id": "ezimuel",
    "name": "Enrico Zimuel"
}
```

```http
GET /speakers HTTP/1.1
Accept: application/json
```
```http
HTTP/1.1 200 OK
Content-Type: application/json
```
```json
{
    "speakers": [
        {
            "id": "ezimuel",
            "name": "Enrico Zimuel"
        },
        {
            "id": "azimuel",
            "name": "Alberto Zimuel"
        },
    ]
}
```

HATEOS - Hypermedia As The Engine Of (application) State
--------------------------------------------------------

* It is the analogous of html links (i.e. links to other resources), but there is not a standard way of implementing this. There are several standards to choose from:
  * HAL
  * JSON-LD
  * Collection+JSON
  * SIREN
  * ...etc

### HAL-JSON - Hypertext Application Language

* supports JSON and XML with the media types `application/hal+json` and `application/hal+xml`
* `_links` is the key that identify an hyperlink

...

