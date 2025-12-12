# RESTful API Design Guidelines

* [Introduction](#introduction)
* [License](#license)
* [Taxonomy](#taxonomy)
* [Our Technology Recommendations for APIs](#our-technology-recommendations-for-apis)
* [Pragmatic REST](#pragmatic-rest)
* [Pragmatic REST Principles](#pragmatic-rest-principles)
* [Resource Oriented Design](#resource-oriented-design)
* [URI Authority Design](#uri-authority-design)
* [Resource Modelling](#resource-modelling)
* [URI Path Design](#uri-path-design)
* [URI Query Design](#uri-query-design)
* [Standard Request Methods](#standard-request-methods)
* [Standard Request Headers](#standard-request-headers)
* [Standard Response Headers](#standard-response-headers)
* [Standard Response Formats](#standard-response-formats)
* [Canonical Data Models](#canonical-data-models)
* [Errors](#errors)
* [Versioning](#versioning)
* [Security](#security)

## Introduction
This document provides guidelines and examples to help developers design simple, consistent and easy-to-use RESTful 
APIs. Much of the material is based on common design challenges faced when developing RESTful web services. Those 
challenges include: resource and representation design; URIs; usage of HTTP; caching; concurrency control; partial 
updates; batch processing; transactions; versioning; and security.

To provide the smoothest possible experience for developers, it's important to have APIs that follow consistent design 
guidelines. Consistency allows teams to leverage common code, patterns, documentation and design decisions.

These guidelines aim to achieve the following:
* Define consistent practices and patterns for all REST APIs.
* Adhere as closely as possible to accepted REST/HTTP good practices.
* Make accessing Services via REST APIs as easy as possible.

### Recommended Reading
Understanding the philosophy behind the 
[REST Architectural Style](https://en.wikipedia.org/wiki/Representational_state_transfer) is essential when designing RESTful APIs.
If you are new to RESTful API design, here are some good resources:
* [REST in Practice](http://shop.oreilly.com/product/9780596805838.do)
* [APIs A Strategy Guide](http://shop.oreilly.com/product/0636920021223.do)
* [REST API Design Rulebook](http://shop.oreilly.com/product/0636920021575.do)
* [RESTful Web Services Cookbook](http://shop.oreilly.com/product/9780596801694.do)
* [Java SOA Cookbook](http://shop.oreilly.com/product/9780596520731.do) (includes a very good discussion of canonical data models and service design)
* [HTTP: The Definitive Guide](http://shop.oreilly.com/product/9781565925090.do)

## License
This work is licensed under the Creative Commons Attribution 3.0 Australia (CC BY 3.0 AU) License. To view a copy of 
this license, visit https://creativecommons.org/licenses/by/3.0/au/.

## Taxonomy

### Errors
Errors, or more specifically Service Errors, are defined as a client passing invalid data to the service and the 
service correctly rejecting that data. Examples include invalid credentials, incorrect parameters, unknown version IDs, 
or similar. These generally result in "4xx" HTTP error codes and are due to client's passing incorrect or invalid data.

### Faults
Service Faults, are defined as the service failing to correctly return a response to a valid client request. These  
generally result in "5xx" HTTP error codes. Calls that fail due to rate limiting do not count as faults. Calls that 
fail as the result of a service fast-failing requests (often for its own protection) do count as faults.

### Latency
Latency is defined as how long a particular API call takes to complete, measured as closely to the client as possible. 
This metric applies to both synchronous and asynchronous APIs in the same way. For long running calls, the latency is 
measured on the initial request and measures how long that call (not the overall operation) takes to complete.

### Time to complete
Services that expose long operations must track "Time to Complete" metrics around those operations.

## Our Technology Recommendations for APIs
The first choices for any API development team today should be:
* Pragmatic REST, because an API should be easy to provide, learn and consume
* JSON, because it is easy to produce and consume
* OAuth, because it prevents password propagation

We describe these technologies as a first choice, not as an only choice. For example, you might decide to include a 
query language, like [GraphQL](https://developer.github.com/v4/guides/) or [falcor](https://netflix.github.io/falcor/), 
with your API.

## Pragmatic REST
The REST architectural style leverages the HTTP protocol to define an approach to inter-computer communications.
It uses URI's to define “resources” and the standard HTTP verbs (POST, GET, PUT, and DELETE) as synonyms for create, 
read, update, and delete (CRUD).

These guidelines seek a balance between a truly RESTful API and a positive developer experience.
We follow REST principles (but not all of them).

## Pragmatic REST Principles

### URIs matter
RESTful APIs use Uniform Resource Identifiers (URIs) to address resources.
And, a good **resource model** makes an API easy to produce, consume, discover, and extend.

### Parameters matter
Use a standard and easy-to-guess set of optional parameters for each API call.

### Data format matters
Make it straightforward for programmers to understand the data the API expects, what kind of data it will return, and 
how to change that.

### Return codes matter
Don't return 200 (OK) when you should be returning 201 (Created) and a
[Location](https://www.w3.org/Protocols/rfc2616/rfc2616-sec14.html#sec14.30) header pointing to the location of the 
new resource.

### Versioning matters
Always version your API. 
It's critical that clients can count on services to be stable over time, and it's critical that services can add 
features and make changes. By default, all requests receive the latest version of the REST API. We encourage you to 
explicitly request a particular version via the Accept Header.

### Everything else should be hidden
Security, rate limiting, routing, and so on can and should be hidden in the HTTP headers.

## Resource Oriented Design
When desiging resource-oriented APIs, follow these steps:
* Determine what types of resources an API provides
* Determine the relationships between resources
* Decide the resource name schemes based on types and relationships
* Decide the resource schemas
* Attach a minimum set of methods to the resources

## URI Authority Design
This section discusses the naming conventions that should be used for the authority portion of a REST API.

### Consistent subdomain names should be used for your APIs
The top-level domain and first subdomain names (e.g., superannuation.ato.gov.au) of an API should identify its service 
owner. The full domain name of an API should add a subdomain named api.

For example:
```
https://api.superannuation.ato.gov.au
```

### Consistent subdomain names should be used for your developer portal
Many REST APIs have an associated website, known as a developer portal, to help onboard new clients with documentation, 
forums, and self-service provisioning of secure API access keys. If an API provides a developer portal, by convention 
it should have a subdomain labeled developer. 

For example:
```
https://developer.superannuation.ato.gov.au
```

Guidelines:
* You should provide a developer portal. See: https://gelato.io/ and https://github.com/Mashape/kong
* You should provide an API status page. See: https://www.statuspage.io/

## Resource Modelling
Resource modelling is an exercise that establishes your API’s key concepts. Before diving directly into the design of
URI paths, it may be helpful to first think about the REST API’s resource model.

### Resource Archetypes
When modelling an API’s resources, we can start with the some basic resource archetypes. Like design patterns, the 
resource archetypes help us consistently communicate the structures and behaviours that are commonly found in REST API 
designs. A REST API is composed of four distinct resource archetypes: **document**, **collection**, **store**, and 
**controller**.

### Document
A document resource is a singular concept that is akin to an object instance or a database record. A document’s state 
representation typically includes both fields with values and links to other related resources. With its fundamental  
field and link-based structure, the document type is the conceptual base archetype of the other resource archetypes.
In other words, the three other resource archetypes can be viewed as specialisations of the document archetype.

Each URI below identifies a document resource:

```
https://api.ato.gov.au/forms/form-1a
https://api.ato.gov.au/fact-sheets/super-and-tax
```

A document may have child resources that represent its specific subordinate concepts. With its ability to bring many 
different resource types together under a single parent, a document is a logical candidate for a REST API’s root 
resource, which is also known as the "service root". The example URI below identifies the service root, which is the 
Superannuation business area's REST API entry point:

```
https://api.superannuation.ato.gov.au
```

The service root should be considered a “namespace”. Namespaces should reflect the consumer's perspective on how a 
resource-oriented API should work, and not necessarily the producer organisations.

### Collection
A collection resource is a server-managed directory of resources. Clients may propose new resources to be added to a 
collection. However, it is up to the collection to choose to create a new resource, or not. A collection resource 
chooses what it wants to contain and also decides the URIs of each contained resource.

Each URI below identifies a collection resource:

```
https://api.superannuation.ato.gov.au/forms
https://api.superannuation.ato.gov.au/fact-sheets
```

See: [sorting](https://github.com/Microsoft/api-guidelines/blob/vNext/Guidelines.md#96-sorting-collections) and 
[filtering](https://github.com/Microsoft/api-guidelines/blob/vNext/Guidelines.md#97-filtering) collections.

### Store
A store is a client-managed resource repository. A store resource lets an API client put resources in, get them back 
out, and decide when to delete them. On their own, stores do not create new resources; therefore a store never 
generates new URIs. Instead, each stored resource has a URI that was chosen by a client when it was initially put into 
the store.

The example interaction below shows a user (with ID 1234) of a client program using a fictional REST API to insert a 
document resource named "receipt-101" in his or her store of deductions:

```
PUT /users/1234/deductions/receipt-101
```

### Controller
A controller resource models a procedural concept. Controller resources are like executable functions, with parameters 
and return values; inputs and outputs. Like a traditional web application’s use of HTML forms, a REST API relies on 
controller resources to perform application-specific actions that cannot be logically mapped to one of the standard 
methods (create, retrieve, update, and delete).

Controller names typically appear as the last segment in a URI path, with no child resources to follow them in the 
hierarchy. The example below shows a controller resource (from a fictional REST API) that allows a client to resend an 
alert to a user:

```
POST /alerts/4321/resend
```

## URI Path Design
Each URI path segment, separated by forward slashes (/), represents a design opportunity. Assigning meaningful values 
to each path segment helps to clearly communicate the hierarchical structure of a REST API’s resource model.

Guidelines:
* A singular noun should be used for document names.
* A (simple) plural noun should be used for collection names (for example, `/partys` not `/parties`).
* A plural noun should be used for store names.
* A verb or verb phrase should be used for controller names.
* Limit your URI's to a maximum of [2048](https://stackoverflow.com/questions/417142/what-is-the-maximum-length-of-a-url-in-different-browsers/417184#417184) characters.

## URI Query Design
A URI’s query comes after the path and before the optional fragment:

```
URI = scheme "://" authority "/" path [ "?" query ] [ "#" fragment ]
```

As a component of a URI, the query contributes to the unique identification of a resource.

The query component can provide clients with additional interaction capabilities such as ad hoc searching and filtering.
Therefore, unlike the other elements of a URI, the query part may be transparent to a REST API’s client.

Guidelines:
* The query component of a URI may be used to filter collections or stores.

See: [sorting](https://github.com/Microsoft/api-guidelines/blob/vNext/Guidelines.md#96-sorting-collections) and 
[filtering](https://github.com/Microsoft/api-guidelines/blob/vNext/Guidelines.md#97-filtering) collections.

## Standard Request Methods
Each HTTP method has specific, well-defined semantics within the context of a REST API’s resource model. Below is a 
list of methods that REST services should support:

| Method  | Description                                                 | Is Idempotent |
| ------- | ----------------------------------------------------------- | ------------- | 
| POST    | Create a new resource in a collection, or submit a command. | **False**     |
| GET     | Retrieve the current representation of a resource.          | True          |
| PUT     | Update a resource, or create a named resource.              | True          |
| DELETE  | Delete a resource.                                          | True          |
| HEAD    | Retrieve GET response headers.                              | True          |
| PATCH   | Apply a partial update to a resource.                       | **False**     |
| OPTIONS | Obtain information about a request.                         | True          |

**POST** should be used to create a new resource within a collection and execute controllers.
**GET** is used to retrieve a representation of a resource’s state. 
**PUT** should be used to add a new resource to a store or update a resource (within a collection).
**DELETE** removes a resource from its parent.
**HEAD** is used to retrieve the metadata associated with the resource’s state.

**Note:** Not all resources will support all methods, but all resources using these methods must conform to their usage.

The recommended usage of the HTTP’s POST method for each of the four resource archetypes:

|     | Document | Collection            | Store | Controller          |
| ----| -------- | --------------------- | ----- | ------------------- |
| POST| _error_  | Create a new resource |_error_| Execute the command |
 
 
## Standard Request Headers 
HTTP defines a set of standard headers, some of which provide information about a requested resource. Other 
headers indicate something about the representation carried by the message. And, a few headers serve as directives to 
control intermediary caches.
 
All header values must follow the syntax rules set forth in the specification where the header field is defined. 
Many HTTP headers are defined in [RFC 7231](https://tools.ietf.org/html/rfc7231), however a complete list of approved 
headers can be found in the [IANA Header Registry](http://www.iana.org/assignments/message-headers/message-headers.xhtml).

Below is a table of request headers that should be used by RESTful API services. Using these headers is not mandated, 
but if used they must be used consistently.

| Header          | Type          | Description                                                                 |
| --------------- | ------------- | --------------------------------------------------------------------------- | 
| Authorization   | String        | Authorisation header for the request.                                       |
| Date            | Date          | Timestamp of the request, in [ISO 8601](https://en.wikipedia.org/wiki/ISO_8601) date and time format. |
| Accept          | Content type  | The requested content type for the response e.g., "application/json".       |
| Accept-Encoding | Gzip, deflate | REST endpoints should support GZIP and DEFLATE encoding.                    |
| Accept-Language | "en", etc.    | Specifies the preferred language for the response.                          |
| Accept-Charset  | "UTF-8"       | The default is UTF-8.                                                       |
| Content-Type    | Content type  | The [mime](https://developer.mozilla.org/en-US/docs/Web/HTTP/Basics_of_HTTP/MIME_types/Complete_list_of_MIME_types) type of request body (POST/PUT/PATCH). |
| Prefer          | return=minimal, return=representation  | If the return=minimal preference is specified, services should return an empty body in response to a successful insert or update. If return=representation is specified, services should return the created or updated resource in the response. |
| If-Match, If-None-Match, If-Range  | String | Services that support updates to resources using optimistic concurrency control must support the If-Match header to do so. Services may also use other headers related to ETags as long as they follow the HTTP specification. |

## Standard Response Headers 
Services should return the following response headers, where required.

| Header             | Type          | Description                                                                 |
| ------------------ | ------------- | --------------------------------------------------------------------------- | 
| Date               | Date          | Timestamp of the response, in [ISO 8601](https://en.wikipedia.org/wiki/ISO_8601) date and time format. |
| Content-Type       | Content type  | The [mime](https://developer.mozilla.org/en-US/docs/Web/HTTP/Basics_of_HTTP/MIME_types/Complete_list_of_MIME_types) type of response body. |
| Content-Encoding   | Gzip, deflate | GZIP or DEFLATE, as appropriate.                                            |
| Preference-Applied | return=minimal, return=representation  | Whether a preference indicated in the Prefer request header was applied. |
| ETag               | String        | The ETag response-header field provides the current value of the entity tag for the requested variant. Used with If-Match, If-None-Match and If-Range to implement optimistic concurrency control. |


**Note:** [RFC 5322](https://tools.ietf.org/html/rfc5322#section-3.3) is a profile of [ISO 8601](https://en.wikipedia.org/wiki/ISO_8601). 

## Standard Response Formats
If an organisation wants to provide the best possible experience for developers, then they must provide data in the 
formats developers are accustomed to using. For example, when a mobile or other low-bandwidth client is involved, 
developers expect JSON message payloads.

Guidelines:
* Services should provide JSON as the default encoding.
* Use camelCase for field names.
* JSON must be well-formed.
* Use [JSON Schema](https://spacetelescope.github.io/understanding-json-schema/index.html) to validate the structure of JSON data.
* XML and other formats may optionally be used for resource representation.

### Pagination
Pagination should be supported using the Link header introduced by [RFC 5988](https://tools.ietf.org/html/rfc5988#page-6).

An API that uses the Link header can return a set of ready-made links so the API consumer doesn't have to construct 
links themselves. This is especially important when pagination is [cursor](https://developers.facebook.com/docs/graph-api/results) based. 

Here is an example of a Link header used properly (taken from GitHub's documentation):

```
Link: <https://api.github.com/user/repos?page=3&per_page=100>; rel="next",
  <https://api.github.com/user/repos?page=50&per_page=100>; rel="last"
```

The possible `rel` values are:

| Name  | Description                                             
| ----- | ------------------------------------------------------------- |
| next  | The link relation for the immediate next page of results.     |
| last  | The link relation for the last page of results.               |
| first | The link relation for the first page of results.              |
| prev  | The link relation for the immediate previous page of results. |


**Note:** If your API requires additional pagination information (like a count of the total number of available results) 
then you can use a custom HTTP header.

## Canonical Data Models
There is a very good discussion of canonical data models and service design in the Eben Hewitt's [Java SOA Cookbook](http://shop.oreilly.com/product/9780596520731.do).
[Enterprise Integration Patterns](http://www.enterpriseintegrationpatterns.com/)) also includes a discussion of canonical data models.

Guidelines:
* Define schemas local to services that reuse a separate layer of schemas, defined independently of services, at the enterprise level.

## Errors
RESTful APIs should use a simple protocol-agnostic error model, which enables a consistent experience across different 
APIs, API protocols, and error contexts (for instance, asynchronous, batch, or workflow errors).

### Error Codes
RESTful APIs should use a standard set of error codes. Individual APIs should avoid defining additional error codes, since 
developers are very unlikely to write logic to handle a large number of error codes. Handing an average of three error 
codes per API call would mean most service logic would just be for error handling, which would not be a good developer 
experience.

A JSON error response:

```
{
  "error": {
    "code": 400,
    "message": "The client specified an invalid argument",
    "status": "INVALID_ARGUMENT",
    "details": [
      {
        "code": "NullValue",
        "target": "familyName",
        "message": "Family name must not be null"
      }
    ]
  }
}

```

HTTP defines forty standard status codes that can be used to convey the results of a client’s request. The status codes 
are divided into five categories:

| Category           | Description                                                                                    |     
| -------------------| ---------------------------------------------------------------------------------------------- |
| 1xx: Informational | Communicates transfer protocol-level information.                                              |
| 2xx: Success       | Indicates that the client’s request was accepted successfully.                                 |
| 3xx: Redirection   | Indicates that the client must take some additional action in order to complete their request. |
| 4xx: Client Error  | This category of error status codes points the finger at clients.                              |
| 5xx: Server Error  | The server takes responsibility for these error status codes.                                  |

Below is a suggested list of standard error codes and a short description of their cause. 
To handle an error, you can check the description for the returned status code and modify your call accordingly.

| HTTP | Type                | Description                                                                 |
| ---  | ------------------- | --------------------------------------------------------------------------- | 
| 200  | OK                  | No Error.                                                                   |
| 400  | INVALID_ARGUMENT    | Client specified an invalid argument. Check error message and error details for more information. |
| 400  | FAILED_PRECONDITION | Request can not be executed in the current system state, such as deleting a non-empty directory. |
| 400  | OUT_OF_RANGE        | Client specified an invalid range.                                          |
| 401  | UNAUTHENTICATED     | Request not authenticated due to missing, invalid, or expired OAuth token.  |
| 403  | PERMISSION_DENIED   | Client does not have sufficient permission.                                 |
| 404  | NOT_FOUND           | The specified resource was not found, or the request is rejected for undisclosed reasons. |
| 409  | ABORTED             | Concurrency conflict, such as read-modify-write conflict.                   |
| 409  | ALREADY_EXISTS      | The resource that a client tried to create already exists.                  |
| 499  | CANCELLED           | Request cancelled by the client.                                            |
| 500  | INTERNAL            | Internal server error. Typically a server bug.                              |
| 501  | NOT_IMPLEMENTED     | API method not implemented by the server.                                   |
| 503  | UNAVAILABLE         | Service unavailable. Typically the server is down.                          |

Guidelines:
* 200 (“OK”) should be used to indicate nonspecific success.
* 201 (“Created”) must be used to indicate successful resource creation.
   The new URI should be returned in the response’s [Location](https://www.w3.org/Protocols/rfc2616/rfc2616-sec14.html#sec14.30) header.
* 202 (“Accepted”) must be used to indicate successful start of an asynchronous action (Controller resources may send 
202 responses, but other resource types should not).
* 301 (“Moved Permanently”) should be used to relocate resources. 
   The new URI should be returned in the response’s [Location](https://www.w3.org/Protocols/rfc2616/rfc2616-sec14.html#sec14.30) header.
* 303 (“See Other”) should be used to refer the client to a different URI.
* 304 (“Not Modified”) should be used to preserve bandwidth.
* 307 (“Temporary Redirect”) should be used to tell clients to resubmit the request to another URI.
* 400 (“Bad Request”) may be used to indicate nonspecific failure
* 403 (“Forbidden”) should be used to forbid access regardless of authorisation state.
* 404 (“Not Found”) must be used when a client’s URI cannot be mapped to a resource.
* 405 (“Method Not Allowed”) must be used when the HTTP method is not supported.
* 406 (“Not Acceptable”) must be used when the requested media type cannot be served.
* 409 (“Conflict”) should be used to indicate a violation of resource state.
* 412 (“Precondition Failed”) should be used to support conditional operations.
* 415 (“Unsupported Media Type”) must be used when the media type of a request’s payload cannot be processed.

### Error Propagation
If your API service depends on other services, you should not blindly propagate errors from those services to your 
clients. When translating errors, we suggest the following:

Guidelines:
* Hide implementation details and confidential information.
* Adjust the party responsible for the error. For example, a server that receives an INVALID_ARGUMENT (400) error from 
another service should propagate an INTERNAL (500) error to its own caller.

## Versioning
A REST API is composed of an assembly of interlinked resources: its resource model. The version of each resource is 
conveyed through its representational form and state.

Guidelines:
* Schemas should be used to manage representational form versions.
* Entity tags should be used to manage representational state versions

## Security
Many RESTful APIs expose resources that are associated with a specific client and/or user. For example, a REST API’s 
documents may contain private information and its controllers may expose operations intended to be executed by a 
restricted audience.

Guidelines:
* API management solutions should be used to protect resources. </br>
  See: [Kong](https://github.com/Mashape/kong), [express-gateway](https://github.com/ExpressGateway/express-gateway) and [node-http-proxy](https://github.com/nodejitsu/node-http-proxy).
* OAuth 2 should be used to protect resources. 
  [OAuth 2](https://oauth.net/2/) uses [Bearer tokens](https://tools.ietf.org/html/rfc6750) and relies on TLS/SSL for transport encryption. </br>
  See: https://getkong.org/plugins/oauth2-authentication/

### Additional Resources
* [The GitHub API v3](https://developer.github.com/v3/)
* [Google Cloud Platform: API Design Guide](https://cloud.google.com/apis/design/)
* [The Microsoft REST API Guidelines](https://github.com/Microsoft/api-guidelines)
* [Vinay Sahni's Best Practices for Designing a Pragmatic RESTful API](http://www.vinaysahni.com/best-practices-for-a-pragmatic-restful-api)
* [The PayPal API Style Guide](https://github.com/paypal/api-standards/blob/master/api-style-guide.md)
* [The Cisco API Design Guide](https://github.com/CiscoDevNet/api-design-guide)
