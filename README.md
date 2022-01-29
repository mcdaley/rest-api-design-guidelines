# Rest API Design Guidelines

The following document provides detailed guidelines and examples for designing 
RESTful Application Programming Interfaces (APIs) that follow the OPEN API 
specification. The goal of the document is to define a set of standards and 
best practices for building APIs, so that product, engineering, and QA teams 
can focus on the business logic of the APIs.

## Rideshare Example
The github repository contains an example Rideshare API that follows the
guidelines and demonstrates how to write an OpenAPI swagger file, see
[taxi.swagger.yaml](./taxi.swagger.yaml). 

## Restful APIs
RESTful APIs leverage the HTTP verbs POST, GET, PUT/PATCH, and DELETE to 
perform CRUD (Create, Read, Update, and Delete) on data or resources. The 
following table shows the expected behavior for typical RESTful APIs and 
the rest of the document will provide details and guidelines for designing 
RESTful APIs.

| Resource                | POST                                   | GET                                     | PUT/PATCH                                 | DELETE                                |
| :---                    | :---                                   | :---                                    | :---                                      | :---                                  |
| /accounts               | Create a new account                   | Retrieve all accounts                   | Bulk update of accounts                   | Remove all accounts                   |
| /accounts/1             | Error                                  | Retrieve details for account 1          | Update details for account if it exists   | Remove account 1                      |
| /accounts/1/transaction | Create a new transaction for account 1 | Retrieve all transactions for account 1 | Bulk update of transactions for account 1 | Remove all transactions for account 1 |

## References
The document leverages the following references:

- [Open API Guide](https://swagger.io/docs/specification/about/)
- [Australian Government - API Design Standard](https://api.gov.au/standards/national_api_standards/index.html)
- [White House Web API Standards](https://github.com/WhiteHouse/api-standards)
- [JSON Schema Specification](https://json-schema.org/specification.html)
- [OData 4.01](https://www.odata.org/documentation/)
- [Standards for Event Driven APIs](https://vedcraft.com/architecture/standards-for-defining-event-driven-and-restful-apis/)
- [Refinery API Standards](https://github.com/refinery29/api-standards#pagination)

# Definitions

## Resource
In order to design a useful and clean API, the system should be broken down 
into logical groupings (often called Models or Resources). In most cases 
the resources are the **nouns** of your system.

For example, in a riedshare application some of the resources are **passengers**, 
**drivers**, **trips**, and **payments**.

## Resource Identifier
Every resource that is available within a system (e.g. every passenger, driver, 
and trip) must be uniquely identifiable within the system. This is a 
key element of the RESTful style of APIs; the ability to individually address 
any item within your system and store these identifiers for later use. Resource 
identifiers can be any of the following:

| Resource Type | Example                                        |
| :---          | :---                                           |
| Numeric       | /customers/43562                               |
| String        | /currency/usd                                  |
| GUID          | /accounts/0d047d80-eb69-4665-9395-6df5a5e569a4 |

The resource identifier **MUST** be unique within the system and **MUST** be 
immutable. ***Primary keys*** or ***Personally Identifiable Information (PII)*** 
**MUST NOT** be exposed. 

## Representation
A key concept in RESTful API design is the idea of the representation 
of a resource at any particular time. When you ask the system for 
passenger information you will receive a representation of the account 
as shown below:

<pre>
  <b>GET /passengers/123456</b>
  200 OK
  Content-Type: application/json

  {
    "first_name":   "John",
    "last_name":    "Doe",
    "mobile":       "415-894-1234",
    "email":        "john.doe@email.com",
    "address": {
      "address_line1":  "1 Market St.",
      "address_line2":  "",
      "city":           "San Francisco",
      "state":          "CA",
      "zip_code":       "94105"
    }
  }
</pre>

This representation can change over time as the system and data within 
changes. A future call to this same endpoint may yield a different 
representation if the passenger changed his email and moved.

<pre>
  <b>GET /passengers/123456</b>
  200 OK
  Content-Type: application/json

  {
    "first_name":   "John",
    "last_name":    "Doe",
    "mobile":       "415-894-1234",
    <b><i>"email":        "johnd@aol.com",</b></i>
    "address": {
      <b><i>"address_line1":  "One Bills Drive",</b></i>
      <b><i>"address_line2":  "",</b></i>
      <b><i>"city":           "Orchard Park",</b></i>
      <b><i>"state":          "NY",</b></i>
      <b><i>"zip_code":       "14075"</b></i>
    }
  }
</pre>

## Controller Resource
A controller resource models a procedural concept. Controller resources are like 
executable functions, with parameters and return values, inputs, and outputs. 
Use a “verb” to denote controller archetype. In the examples below, the controller 
resource for trips is for the passenger to pay for the trip after being dropped off.

```
// Pay for a trip after dropoff
POST /trips/{tripId}/pay
```

## Namespace
A namespace is an optional way for a service to define a grouping of 
related functions within the system. Namespaces could be high-level 
such as a cobrand-1 or low-level such as loans or accounts.

# Naming Conventions

## Message Format
Request and response body field names and query parameter names MUST 
follow consistent naming conventions, and SHOULD be either camelCase OR 
snake_case as shown below:

```
// thisIsCamelCase
{
  “customerId”: “AB1837”
}
```

```
// this_is_snake_case
{
  “customer_id”: “AB1837”
}
```

Fields that represent arrays SHOULD be named using plural nouns (e.g.accounts - 
contains one or more accounts). The object and field definition MUST be the 
same for the request and response body as well as corresponding query parameter 
values.

We will use snake_case as its standard naming convention for the rest of the document.

## Uniform Resource Identifier (URI)
Rest APIs use Uniform Resource Identifiers (URIs) to access resources and the URI structure is:

```
https://[host]:[port]/namespace/api/v{version_number}/{resource}?{query_param1=x}
```

Where the URI has the following variable definitions:

| Variable          | Definition |
| :---              | :---       |
| protocol          | All APIs MUST be exposed using HTTPS. |
| host              | Hostname or IP address of the application, such as 127.0.0.1 or www.gadgets.com |
| port              | TCP Port to connect to the API, default is 3000 and is typically not required in production APIs. |
| namespace         | Optional field for grouping resources. |
| version_number    | Major version number of the API, see Versioning for more details. |
| resource          | A resource is any kind of object, data, or service that can be accessed by the client, such as books, accounts, or animals. Resources should always be plural nouns. |
| query_param=value | Optional list of query parameters to filter the resulting data set. For example the following would sort a list of resources by descending date, ?sort_fields=date&sort=desc. Query parameters MUST NOT be used to transport payload or actual data. |

### URI Maximum Length
The total URI, including the Path and the Query, MUST NOT exceed 2000 characters 
in length including any formatting codes such as commas, underscores, question 
marks, hyphens, plus or slashes.

### URIs MUST follow the standard naming convention as described below:

- The URI MUST be specified in all lower case
- Only hyphens ‘-’ can be used to separate words or path parameters for readability (no spaces or underscores)
- Underscores ‘_’ or camelCase can be used to separate words in query parameters, but not as part of the base URI.

The following sections defines rules for naming URI resources, resource 
identifiers, and query parameters

#### Resource
The resource name in the URI should always be a plural noun and is written in 
lower case and use “dashes” between words when the resource consists of more 
than one word., e.g., “team-members, accounts”

The following are examples of resources:

```
GET /api/v1/drivers
GET /api/v1/credit-cards
```

**Note:** This is the only place where hyphens are used as a word separator. 
In nearly all other situations camelCase OR snake_case, ( _ ), MUST be used.

#### Resource Identifier
When performing an action on a specific resource then the Id in the documentation 
will be the name of the resource w/ ‘_id’ appended to the end, e.g. **passenger_id**, 
**trip_id** when using the snake_case naming convention. The following are examples 
of naming Ids:

```
GET /api/v1/drivers/{driver_id}
GET /api/v1/drivers/{driver_id_id}/trips/{trip_id}
```

#### Query Parameters
URIs to access Rest APIs may support the ability to filter the result set by 
adding query parameters at the end of the URI as shown previously. Query 
parameters will be: 

- Optional.
- All lowercase with words separated by underscores (i.e. start_date).
- MUST start with a letter and SHOULD be either camelCase or snake_case, consistent with the case standard employed for field names.
- MUST be percent-encoded. For example, AWS requires query parameter names to conform to the regex ^[a-zA-Z0-9._$-]+$.
- SHOULD not contain characters that are not URL safe.

## Hierarchical Data URIs 
Resources often have some kind of functional hierarchy and it makes sense to 
nest the resources in the URI, for example to get the payments to a driver:

```
// Return all payments for a driver
GET /drivers/{driver_id}/payments

// Return a single payment for a driver
GET /drivers/{driver_id}/payments/{payment_id}
```

Using the functional hierarchy is recommended over adding query parameters, 
for example to get all of a driver’s payments do NOT make the URI: 

```
GET /payments?driver_id=1234. 
```

Typically, we do not want to nest too many levels in the hierarchy and it is 
recommended to stop after one level.

## Example URIs

### Good URIs

```
// List of magazines
GET https://www.ex.com/api/v1/magazines

// Sort list of magazines by issue date from earliest to latest
GET https://www.ex.com/api/v1/magazines?sort_field=issue_date&sort=asc

// Get a specific issue of a magazine
GET https://www.ex.com/api/v1/magazines/34

// All articles in an issue of a magazine
GET https://www.ex.com/api/v1/magazines/34/articles

// Get a specific article in a magazine
GET https://www.ex.com/api/v1/magazines/34/articles/99
```

### Bad URIs

```
// Non-plural resource:
GET http://www.example.com/api/v1/magazine
GET http://www.example.com/api/v1/magazine/34

// Verb in URL
POST http://www.example.com/api/v1/magazines/35/create

// Filter outside of query string
GET http://www.example.com/api/v1/magazines/2020/desc
```

# Data Types
The [Open API Specification](https://swagger.io/docs/specification/data-models/data-types/) 
defines all of the data types supported by APIs that includes strings, numbers, 
integers, boolean, array, and objects. The following sections highlight the 
supported data types in the request and response bodies and query parameters.

## Numbers
| Type    | Format | Description |
| :---    | :---   | :---        |
| number  | -      | Any numbers |
| number  | float  | Floating-point numbers |
| number  | double | Floating-point numbers with double precision |
| integer | -      | Integer numbers |
| integer | int32  | Signed 32-bit integers (commonly used integer type) |
| integer | int64  | Signed 64-bit integers (long type) |

**Note:** that strings containing numbers, such as “17” are considered 
strings and not numbers.

## Boolean
The boolean type represents true and false.

## String
Strings are used to specify text values such as email, uuid, and any other 
text values. The OPEN API Specification also defines a format modifier 
for string formats for date, date-time, password, byte, and binary.

## Dates
Dates are formatted strings that can be converted to the Date and Datetime 
by API clients and the following sections define the standard formats.

### Date
In the request body, response body, and query parameters dates will be strings 
formatted using the `YYYY-MM-DD` format (e.g., 2021-09-18).

### Datetime
In the request body, response body, and query parameters datetime values will 
be strings formatted as:

```
YYYY-MM-DDThh:mm:ss:fff
```

Where,

| Date Component | Description | Example |
| :---           | :---        | :---    |
| YYYY           | Four digit year                      | 2021 |
| MM             | Two digit month (with leading zero)  | 04 |
| DD             | Two digit date (with leading zero)   | 27 |
| T              | Set character indicating the start of the time element in a datetime format | T |
| hh             | Two digits hour (00 - 23)            | 18 |
| mm             | Two digits minute (00-59)            | 38 |
| ss             | Two digits second | 10 |
| fff            | Three digits millisecond (000 - 999) | 456 |

### Time Zones
TBD

### Date Field Naming Conventions
When using data fields, the following naming conventions for these fields 
should be used:

- For properties requiring both date and time, services MUST use the suffix datetime, e.g., `start_datetime`.
- For properties requiring only date information, services MUST use the suffix date, e.g., `birth_date`.
- For properties requiring only time information, services MUST use the suffix time, e.g., `start_time`.

## Money
TBD
Monetary values need to be stored in a number format that will not improperly 
round up or round down values.

## Currency
Use the 3 character ISO definitions for the currency codes, i.e., USD, EUR.

### Country Codes
Use the 3 character ISO definitions for country codes, i.e., USA, CAN, GER.

## Enumerated Values
For all enumerated values the APIs will define human understandable string 
definitions for request, response, and URI parameters instead of numeric 
values. For example account type would equal ‘Checking’ or ‘Savings’ instead 
of 1001 or 1002. The application is responsible for mapping the human readable
enum strings to their corresponding codes if that is how the application stores
the enumerated values.

## Addresses
An address is a system resource and the following format can be used for
US addresses and is based on lob.com APIs which is a direct mail service:

| Field   | Required | Format                         | Description |
| :---    | :---     | :---                           | :--- |
| name    | Yes      | <= 40 characters               |
| company | No       | <= 40 characters               | Optional company name for the user. |
| line1   | Yes      | <= 64 characters               | The primary number, street name, and directional information. |
| line2   | No       | <= 64 characters               | An optional field containing any information which can't fit into line 1. |
| city    | Yes      | <= 200 characters              | |
| state   | Yes      | 2 letter state short-name code | Enumerated list of 50 states, e.g. CA, NY, ... |
| zip     | Yes      | ^\d{5}(-\d{4})?$               | Must follow the ZIP format of 12345 or ZIP+4 format of 12345-1234. |
| country | Yes      | 3 letter ISO country code.     | Enumerated list of country codes, e.g., USA, CAN |

# API Versioning
An API is a public contract between a server and a consumer and it is necessary 
to support old versions of the API when new versions are released.  APIs 
need to follow Semantic Versioning [https://semver.org](https://semver.org). The 
version number for the APIs will support MAJOR.MINOR.PATCH versions and the 
versions are incremented:

| Version | Description |
| :---    | :--- |
| MAJOR   | When incompatible API changes are made to the API. |
| MINOR   | When functionality is added that is backwards compatible. |
| PATCH   | When backward compatible bug fixes are implemented. |

**NOTE:** Only the MAJOR version is specified in the URI and responses will
include the version number of the API to help developers debug issues.

# API Requests

## Request Headers
APIs can support various request headers and the following sections define
some of the common request headers:

### Authorization
APIs Needs to support one of the following authorization headers:
- API Key
- Basic Auth (API Key + Secret)
- Bear {json web token}	

### Content-Type
Type of content to send, a choice of:
- application/json
- application/xml
- application/x-www-form-urlencoded
- multipart/form-data
- text/plain; charset=utf-8
- text/html
- application/pdf
- image/png

### Accept
Type of content to send, a choice of:
- application/json
- application/xml
- application/x-www-form-urlencoded
- multipart/form-data
- text/plain; charset=utf-8
- text/html
- application/pdf
- image/png

### Request-Id
Unique identifier provided by the client that allows the service to track 
client requests and to support idempotency.

### Connection
TBD

### Date
The date and time at which the message was originated, in "HTTP-date" format 
as defined by RFC 7231 Date/Time Formats. E.g. Tue, 15 Jan 2022 08:12:31 GMT

### Cache-Control
TBD

Payload data **MUST NOT** be used in HTTP Headers. They are reserved for 
transversal information (authentication token, monitoring token, request 
properties etc).

## HTTP Request Methods
Rest APIs leverage the Http  **POST**, **GET**, **PUT/PATCH**, and **DELETE** 
operations to perform CRUD (Create, Read, Update, and Delete) on namespaces, 
resources, and resource identifiers, where an an operation is defined by the 
user of a HTTP method and a resource path. The following table provides 
definitions and examples of HTTP requests:

### GET
Fetches one or more resources.

```
GET /passengers
GET /drivers
GET /drivers/456789
GET /drivers/456789/payments
GET /drivers/456789/payments/123
```

### POST
Creates a new resource.

```
POST /trips
POST /passengers
POST /passengers/456789/credit-cards
```

### PUT
Updates all of the fields in a resource.

```
PUT /drivers/456789
PUT /passengers/john-doe
PUT /passengers/john-doe/credit-cards/123
```

### PATCH
Update some of the fields in a resource.

```
PATCH /trips/456789
PATCH /passengers/john-doe
PATCH /accounts/john-doe/addresses/123
```

### DELETE
Deletes a resource.

```
DELETE /drivers/456789
DELETE /passengers/john-doe
DELETE /passengers/john-doe/credit-cards/123
```

## Request Payload Formats
The APIs MUST support a JSON formatted payload when supplied. Other payload formats 
such as XML, CSB, and YAML may be supported as required. The JSON payloads need 
to follow the [JSON Schema Specification](https://json-schema.org/specification.html), 
below is an example JSON request payload.

```
{
  “field1”: “value1”,
  “field2”: “value2”,
  “field3”: “value3”,
  “field4”: {
    “subfield1”: “subvalue1”,
    “subfield2”: “subvalue2”
  }
}
```

## Idempotency
An idempotent HTTP method is one that can be called many times without different 
outcomes. In some cases, secondary calls will result in a different response 
code, but there will be no change in the state of the resource.

As an example, when you invoke N similar DELETE requests, the first request will 
delete the resource and the response will be 200 (OK) or 204 (No Content). Further 
requests will return 404 (Not Found). Clearly, the response is different from 
the first request, but there is no change of state for any resource on the server 
side because the original resource is already deleted.

RESTful API methods MUST adhere to the specified idempotency defined in the 
table below:

| HTTP Method | Is  Idempotent? |
| :---        | :---            |
| GET         | True            |
| POST        | False           |
| PUT         | True            |
| PATCH       | False           |
| DELETE      | True            |
| HEAD        | True            |
| OPTIONS     | True            |

# Query Parameters
RESTful APIs may support the ability to filter the result set of resources by 
adding query parameters to the end of the URI. The sections below define the 
standard URI query parameters.

## Pagination
Pagination is the process of returning a large set of results in chunks (or pages) 
to reduce the amount of information that is sent with each response. Pagination 
can be implemented using offset-based or cursor-based pagination and this document
will cover offset pagination.

### Offset Pagination
The following query parameters are used to support offset-pagination.

#### page
The page that the user wants to return, for example:

```
page=1
```

Where the default page is 1.

#### page_size
The number of results per page the user wants to return, for example:

```
page_size=10
```

Where the default page_size is 25 and the maximum is 100.

### Cursor Pagination
In cursor-based pagination the API response contains a cursor that can be
used to retrieve the next page of results.

## Sorting
Sorting allows the user to select the order of the collection of resources 
returned in the API response. For example, users could sort a list of transactions 
by descending date or by description alphabetically. The two parameters to 
define sorting are:

### sort
The direction to sort the resources, the supported values are **asc** and 
**desc** and the default direction is **asc**.

### sort_fields
The fields to sort the resources, for example to sort by date and name add 
the following query parameters:

```
?sort_fields=date,name&sort=desc
```

### sort_by
If it is required to sort by multiple fields in ascending and descending order
then the sort_by option can be used. To sort by descending date and ascending 
name then the following options can be used:

```
sort_by=-date,+name

or

sort_by=date:desc,name:asc
```

If **asc** is the default sort order then the "+" and "asc" are not required.

## Filtering
Filtering is used to control which resources and which fields in the resource 
are returned in the response.

### Simple Filtering
Attributes can be used to filter a collection of resources, for example:

```
?last_name=Citizen
```

filters out the collection of resources with the attribute last_name that matches 
Citizen. APIs may also support multiple filters:

```
?last_name=Citizen&date_of_birth=1999-12-31
```

Filters out the collection of resources with the attribute last_name that matches 
Citizen and date_of_birth that matches 31st of December, 1999.

The equal (=) operator is the only supported operator when used in this technique. 
For other operators and conditions see Advanced Filtering.

### Advanced Filtering
There are situations where simple filtering does not meet the needs and a more comprehensive approach is required. Use the reserved keyword 'filter' to define a more complex filtering logic.

Complex filter logic can be chained together in a single 'filter' value, using OData 4.0 
query compliant query strings. The following operators should supported at a minimum:

1. gt - Greater than
2. lt - Less than
3. ge - Greater than or equal to
4. le - Less than or equal to
5. eq - Equal
6. ne - Not equal

The ‘and’ and ‘or’ conditions should be supported for compound filtering, as shown in 
the example below:

```
?filter=creation_date gt 2001-09-20T13:00:00 and creation_date lt 2001-09-21T13:00:00 and post_code eq 94105
```

Returns a collection of resources where the creation_date is between 2001-09-20 1pm 
and 2001-09-21 1pm and post_code is 94105.

### Output Selection
The fields attribute can be used to specify the fields to return in the response 
payload. The example below returns only the first_name and last_name fields in 
the response:

```
?fields=first_name,last_name
```

# API Responses

## Response Headers
APIs need to support the following response headers:

### Content-Type
Type of content in the response body, a choice of:
- application/json
- application/pdf

### Cache-Control
Informs the caching mechanisms. It is measured in seconds.max-age=3600

### Date
The date and time at which the message was originated, in "HTTP-date" format 
as defined by RFC 7231 Date/Time Formats. E.g. Tue, 15 Nov 2021 08:12:31 GMT

### Expires
Gives the date/time after which the response is considered stale e.g. 
Thu, 01 Dec 2021 16:00:00 GMT

## API-Version
Version of the API that processed the request and will be in the MAJOR.MINOR.PATCH format.

## HTTP Response Codes
The API must return standard HTTP response codes as defined in 
[Mozilla Developer Network](https://developer.mozilla.org/en-US/docs/Web/HTTP/Status) and 
the possible response codes for each type of request are defined in the REST API 
Standard Request and Responses section of the document.

## Response Payload
The RESTful APIs will return JSON responses and the following sections show 
example payloads for a response that returns a single resource and for a 
response that returns a collection of resources.

### Metadata
A response may contain an optional “metadata” section that provides information 
about the response. For example, when requesting a list of resources the metadata 
section may contain information needed by the developer to implement 
pagination as shown below:

```
{
  “metadata”: {
    "pagination": {
      “total_count”: 100,
      “page”:        2,
      “page_size”:   25
    }
  },
  “{resources}”: [
    ...
  ]
}
```

### Single Resource
A response with a single resource returns the resource name and all of the key/value 
pairs of the resource.

```
{
  “{resource}”: {
    “id”:         “{uniqueId}”,
    “field1”:     “value1”,
    “field2”:     “value2”,
    “field3”:     “value3”,
    “created_at”: “datetime”,
    “updated_at”: “datetime”,
  }
}
```

### Collection of Resources
A response with a collection of resources returns the resource name and the array 
of resources fetched in response.

```
{
  “{resources}”: [
    {
      “id”:     “uuidv4”,
      “field1”: “value1”,
      “field2”: “value2”,
      “field3”: “value3”,
      “field4”: {
        “subfield1”: “subvalue1”,
        “subfield2”: “subvalue2”
      },
      “createdAt”:    “datetime”,
      “lastModified”: “datetime”
    },
    {
      “id”:     “uuidv4”,
      “field1”: “value1”,
      “field2”: “value2”,
      “field3”: “value3”,
      “field4”: {
        “subfield1”: “subvalue1”,
        “subfield2”: “subvalue2”
      },
      “created_at”: “{datetime}”,
      “updated_at”: “{datetime}”
    },
    ...
  ],
  “metadata”: {
    "pagination": {
      “total_count”: 100,
      “page”:        2,
      “page_size”:   25
    }
  }
}
```

# Error Handling
The purpose of this section is to define a set of consistent guidelines for 
defining and handling application errors in order to provide a consistent 
experience for developers working with the APIs and for internal Product, 
Development, and QA teams in implementing error management in the APIs. The 
Error response returns an array of error messages, so that it can handle 
cases where there are multiple errors. Error messages should have the 
following fields:

| Error Attribute | Mandatory | Description |
| :---            | :---      | :--- |
| request_id      | Mandatory | Unique API request identifier for the specific error that can be searched for in the application log files. |
| httpStatus      | Optional  | HTTP Status of the response. |
| code            | Mandatory | An application specific numeric error code |
| resource        | Mandatory | The resource where the error occurred. |
| title           | Mandatory | The error’s title |
| message         | Mandatory | Text that specifies the error, for example **Passenger Id “456789” is not found**. |
| stack           | Optional  | A stack trace of the error that can be used in development and testing. |

Below is an example error message:

```
{
  “errors”: [{
    “httpStatus”: 404,
    “request_id”: “86032cbe-a804-4c3b-86ce-ec3041e3effc”,
    “code”:       1101,
    “resource”:   “Drivers”,
    “title”:      “Resource Not Found”,
    “message”:    “Driver Id “xys786” is not found”
  }]
}
```

# Logging
Logging helps developers find bugs in the APIs and helps customer support to 
resolve client issues. Log messages should be JSON formatted so that tools 
can be used to aggregate logs and parse the log messages. Each log message 
should have the following the fields:

| Field           | Description |
| :---            | :---        |
| Log Level       | Debug, Info, Warn, Error, or Fatal |
| Timestamp       | UTC Timestamp of the log message |
| Host/IP Address | Server running the application. |
| Service         | Name of the microservice logging the message, e.g., auth, payments, … |
| Request Id      | Unique identifier used to track the API request through multiple microservices. |
| Client/User Id  | Unique identifier to track which user/client made the API request. |
| Message         | Human understandable message that describes the informational or error message. |

Below is an example JSON log message stating that a trip was created:

```
{"timestamp":"2021-08-06 09:28:52", "level":"info", "service":"trip", 
"ip":"127.0.0.1", "requestId":"xyz","userId":"marv@bills.com",
"message":"Created trip w/ id=610d6344989f17053e69e8ff"}
```

# Event Notification
TBD

# Security
TBD

# REST API Standard Request and Responses
The following sections define standard RESTful API requests and responses using an 
example banking system. The examples cover all of the basic CRUD operations for a 
RESTful API and define when to use different response codes.

**Note:** The API Responses may seem verbose by returning
the resource instead of less information, but the idea is to standardize 
and simplify API definitions and coding standards as will be shown in example
code for each of the responses.


## Create Resource: POST /api/v1/{resource}
The ```POST /api/v1/{resource}``` creates a new resource and returns the new 
resource in the response.

### Request

#### Query Parameters
NA

#### Headers
| Header       | Definition |
| :---         | :--- |
| Content Type | application/json |
| Authorize    | Bearer {JWT} |
| requestID    | Unique Id for the transaction to support idempotency |

#### Body
Example request body for creating a new Driver:

```
{
  "first_name":   "Alex",
  "last_name":    "Reiger",
  "email":        "alex.reiger@taxi.com",
  "mobile":       "202-875-4343",
  "driver_license": {
    "license_number":   "CA5260863502",
    "name":             "Alex Reiger",
    "state":            "CA",
    "expiration_date":  "2024-03-17"
  }
}
```

### Response

#### Http Status
| Code | Status                 | When To Use |
| :--- | :---                   | :--- |
| 201  | Created                | The resource was created. The Response Location HTTP header SHOULD be returned to indicate where the newly created resource is accessible. |
| 202  | Accepted               | Is used for asynchronous processing to indicate that the server has accepted the request but the result is not available yet. The Response Location HTTP header may be returned to indicate where the created resource will be accessible. |
| 400  | Bad Request            | The server cannot process the request (such as malformed request syntax, size too large, invalid request message framing, or deceptive request routing, invalid values in the request) For example, the API requires a numerical identifier and the client sent a text value instead, the server will return this status code. |
| 401  | Unauthorized           | The request could not be authenticated. |
| 403  | Forbidden              | The request was authenticated but it is not authorized to access the resource. |
| 405  | Not Allowed            | The method is not implemented for this resource. The response may include an Allow header containing a list of valid methods for the resource. |
| 415  | Unsupported Media Type | This status code indicates that the server refuses to accept the request because the content type specified in the request is not supported by the server. |
| 422  | Unprocessable Entity   | This status code indicates that the server received the request but it did not fulfil the requirements of the back end. An example is a mandatory field was not provided in the payload. |
| 500  | Internal Server Error  | An internal server error. |

#### Headers
| Header      | Definition |
| :---        | :---       |
| Accept Type | application/json |
| Api Version | {major}.{minor}.{patch} |
| requestId   | Unique identifier sent in the client request for idempotency |

#### Body
The body of the create resource action always returns the resource that was created 
and it will be under the resource name section for the response body. For example, 
a new account will be in the “account” field in the response and each new resource 
will generate a unique Id and add createdAt and updatedAt timestamps to the 
resource as shown in the example below:

```
{
  "driver": {
    "id":           "d68825b3-830c-44f8-953e-a8b5cd943aa8",
    "first_name":   "Alex",
    "last_name":    "Reiger",
    "email":        "alex.reiger@taxi.com",
    "mobile":       "202-875-4343",
    "driver_license": {
      "id":               "671c272f-370f-40f9-b863-7531aa696b4a",
      "license_number":   "CA5260863502",
      "name":             "Alex Reiger",
      "state":            "CA",
      "expiration_date":  "2024-03-17"
    },
    “created_at”:   “2021-09-14T22:35:56.560Z”,
    “updated_at”:   “2021-09-14T22:35:56.560Z”,
  }
}
```

Having the resource name in the response makes it easier for developers
to parse the response. For example in javascript a developer can access 
the new account using the code:

```javascript
const { driver }   = response.body
console.log(`Driver name=${driver.last_name}, mobile=${driver.mobile})
```

## Fetch List of Resources: GET /api/v1/{resources}
The ```GET /api/v1/{resources}``` returns an array of all the resources and 
also a metadata section that provides detail about paging the results.

### Request

#### Query Parameters
The ```GET /api/v1/{resources}``` supports a set of standard query parameters that 
allows users to control the data returned by the API. The user can specify the 
following parameters:

##### page_size
Max number of resources to return, the API should set a default page_size so 
that APIs do not return an infinite amount of resources.

##### page
The page of the data to return and the default is 1. For example, if the 
page_size = 25 and page = 2 then the API returns records 25 - 49.

##### sort_by
Specify the sort fields and order. For example to sort by descending date and
ascending amount:

```
sort_by=date:desc,amount:asc
```

##### fields
Filter the fields returned in the result. For example to return only the date,
description, and amount for all transactions:

```
fields=date,description,amount
```

#### Headers
| Header       | Definition |
| :---         | :--- |
| Content Type | application/json |
| Authorize    | Bearer {JWT} |
| requestID    | Unique Id for logging the request |

#### Body
NA

### Response

#### Http Status
| Code | Status                 | When To Use |
| :--- | :---                   | :--- | 
| 200  | Ok                     | The request was successfully processed. |
| 401  | Unauthorized           | The request could not be authenticated. |
| 403  | Forbidden              | The request was authenticated but it is not authorized to access the resource. |
| 405  | Not Allowed            | The method is not implemented for this resource. The response may include an Allow header containing a list of valid methods for the resource. |
| 500  | Internal Server Error  | An internal server error. |


#### Headers
| Header      | Definition |
| :---        | :---       |
| Accept Type | application/json |
| Api Version | {major}.{minor}.{patch} |
| requestId   | Unique identifier sent in the client request for logging |

#### Body
The response body to get a collection of resources returns the resources defined by 
the plural resource name and an array of resources. In the example below the 
resources are defined by the “trips” label. The response also includes 
data needed to define pagination in the “metadata” section.

```
{
  "trips": [
    {
      "id":             "beaf7b93-b8de-41eb-92af-8af58b50409b",
      "passenger_id":   "e614cc4b-9bd7-47c2-85b3-eca88cb9236e",
      "driver_id":      "1ade3c8b-f30d-4bca-8cd8-cd123d4906e9",
      "pickup_location": {
        "address_line1":  "1 Market St.",
        "city":           "San Francisco",
        "state":          "CA",
        "zip_code":       94105
      },
      "dropoff_location": {
        "address_line1":  "1705 Mariposa St",
        "city":           "San Francisco",
        "state":          "CA",
        "zip_code":       94107
      },
      "request_datetime": "string",
      "status":           "Accepted",
      "amount":           25.00
    },
    ...
  ],
  "metadata": {
    "pagination": {
      "page":         2,
      "page_size":    20,
      "total_count":  155
    }
  }
}
```

Organizing the response by the plural resource makes it easy for a 
developer to process the response and the pagination data. For 
example, to process all the transactions in the response in JavaScript:

```javascript
const { trips }       = response.body
const { pagination }  = response.body.metadata

trips.forEach( (trip) => {
  console.log(`Trip id=${trip.id}, amount=${trip.amount})
})
```

## Fetch Single Resource: GET /api/v1/{resources}/{resource_id}
The ```GET /api/v1/{resources}/{resource_id}``` returns a single resource. If 
the resource is not found then it returns a 404 Not Found error.

### Request

#### Query Parameters
NA

#### Headers
| Header       | Definition |
| :---         | :--- |
| Content Type | application/json |
| Authorize    | Bearer {JWT} |
| requestID    | Unique Id for logging the request |

#### Body
NA

### Response

#### Http Status
| Code | Status                 | When To Use |
| :--- | :---                   | :--- | 
| 200  | Ok                     | The request was successfully processed. |
| 401  | Unauthorized           | The request could not be authenticated. |
| 403  | Forbidden              | The request was authenticated but it is not authorized to access the resource. |
| 405  | Not Allowed            | The method is not implemented for this resource. The response may include an Allow header containing a list of valid methods for the resource. |
| 500  | Internal Server Error  | An internal server error. |


#### Headers
| Header      | Definition |
| :---        | :---       |
| Accept Type | application/json |
| Api Version | {major}.{minor}.{patch} |
| requestId   | Unique identifier sent in the client request for logging |

#### Body
The following response retrieved a single passenger from the taxi app.

```
{
  "passenger": {
    "id":         "d290f1ee-6c54-4b01-90e6-d701748f0851",
    "first_name": "John",
    "last_name":  "Doe",
    "mobile":     "415-894-1234",
    "email":      "john.doe@email.com",
    "address": {
      "id":             "79455671-2e46-46f1-b046-37d827f33103",
      "address_line1":  "1 Market St.",
      "city":           "San Francisco",
      "state":          "CA",
      "zip_code":       94105
    }
  }
}
```

Organizing the response by the resource makes it easy for a developer 
to process the response. For example to process the transaction in 
JavaScript:

```javascript
const { passenger } = response.body
console.log(`Passenger first_name=${passenger.first_name} mobile=${passenger.mobile}`)
```

## Update Resource Attributes: PATCH /api/v1/{resources}/{resource_id}
The ```PATCH /api/v1/{resources}/{resource_id}``` updates a resource and 
returns the updated resource in the response. Another option is to return
the resource_id of the updated resource instead of the whole resource. PATCH 
is used when updating some of the fields of a resource and it is not idempotent.

### Request

#### Query Parameters

##### resource
If set to true return the updated resource otherwise return the updated 
resource id. API can set the default to true or false.

#### Headers
| Header       | Definition |
| :---         | :--- |
| Content Type | application/json |
| Authorize    | Bearer {JWT} |
| requestID    | Unique identifier sent in the client request for idempotency. |

#### Body
The following example code updates the passengers mobile number and email.

```
{
  mobile: 510-455-1515
  email:  johnd@gmail.com
}
```

### Response

#### Http Status
| Code | Status                 | When To Use |
| :--- | :---                   | :--- | 
| 200  | Ok                     | The request was successfully processed. |
| 400  | Bad Request            | The server cannot process the request (such as malformed request syntax, size too large, invalid request message framing, or deceptive request routing, invalid values in the request). For example, the API requires a numerical identifier and the client sends a text value instead, the server will return this status code. |
| 401  | Unauthorized           | The request could not be authenticated. |
| 403  | Forbidden              | The request was authenticated but it is not authorized to access the resource. |
| 405  | Not Allowed            | The method is not implemented for this resource. The response may include an Allow header containing a list of valid methods for the resource. |
| 500  | Internal Server Error  | An internal server error. |


#### Headers
| Header      | Definition |
| :---        | :---       |
| Accept Type | application/json |
| Api Version | {major}.{minor}.{patch} |
| requestId   | Unique identifier sent in the client request for idempotency. |

#### Body
There are 2 alternatives when building the PATCH response. The first alternative
is to return the resource_id with the updated_at timestamp that validates the
resource was updated as shown below:

```
{
  id:         "1d973f86-690e-4632-ba01-fee2580b0264",
  updated_at: "2022-01-14T22:35:56.560Z"
}
```

The second option is to return the updated resource as shown below:

```
{
  "passenger": {
    "id":         "d290f1ee-6c54-4b01-90e6-d701748f0851",
    "first_name": "John",
    "last_name":  "Doe",
    mobile:       510-455-1515
    email:        johnd@gmail.com
    "address": {
      "id":             "79455671-2e46-46f1-b046-37d827f33103",
      "address_line1":  "1 Market St.",
      "city":           "San Francisco",
      "state":          "CA",
      "zip_code":       94105
    }
  }
}
```

## Update Resource: PUT /api/v1/{resources}/{resource_id}
The ```PUT /api/v1/{resources}/{resource_id}``` replaces an existing
resource with a new resource and requires that all of the resource 
fields are included in the request. Unlike a PATCH request, a PUT 
request is idempotent.

### Request

#### Query Parameters

##### resource  
If set to true return the updated resource otherwise return the updated 
resource id. API can set the default to true or false.

#### Headers
| Header       | Definition |
| :---         | :--- |
| Content Type | application/json |
| Authorize    | Bearer {JWT} |
| requestID    | Unique identifier sent in the client request for logging. |

#### Body
The following example code updates a passenger's address when the passenger 
moves. For the PUT request all of the address fields are required in the request.

```
{
  "name":   "John Doe",
  "line1":  "1 Bills Dr.",
  "line2":  "",
  "city":   "Orchard Park",
  "state":  "NY",
  "zip":    "14127"
}
```

### Response

#### Http Status
| Code | Status                 | When To Use |
| :--- | :---                   | :--- | 
| 200  | Ok                     | The request was successfully processed. |
| 400  | Bad Request            | The server cannot process the request (such as malformed request syntax, size too large, invalid request message framing, or deceptive request routing, invalid values in the request). For example, the API requires a numerical identifier and the client sends a text value instead, the server will return this status code. |
| 401  | Unauthorized           | The request could not be authenticated. |
| 403  | Forbidden              | The request was authenticated but it is not authorized to access the resource. |
| 405  | Not Allowed            | The method is not implemented for this resource. The response may include an Allow header containing a list of valid methods for the resource. |
| 500  | Internal Server Error  | An internal server error. |


#### Headers
| Header      | Definition |
| :---        | :---       |
| Accept Type | application/json |
| Api Version | {major}.{minor}.{patch} |
| requestId   | Unique identifier sent in the client request for loggin. |

#### Body
There are 2 alternatives when building the PUT response. The first alternative
is to return the resource_id with the updated_at timestamp that validates the
resource was updated as shown below:

```
{
  id:         "e738f1a9-4012-4866-8031-5701d6a13e2a",
  updated_at: "2022-01-14T22:35:56.560Z"
}
```

The second option is to return the updated resource as shown below:

```
{
  “address”: {
    “id”:           “e738f1a9-4012-4866-8031-5701d6a13e2a”,
    "name":         "John Doe",
    "line1":        "1 Bills Dr.",
    "line2":        "",
    "city":         "Orchard Park",
    "state":        "NY",
    "zip":          "14127"
    “created_at”:   “2021-09-14T22:35:56.560Z”,
    “updated_at”:   “2022-01-14T22:35:56.560Z”,
  }
}
```

## Delete a Resource: DELETE /api/v1/{resources}/{resource_id}
The ```DELETE /api/v1/{resources}/{resource_id}``` deletes the resource 
specified by the resource_id and returns an empty message with a Http 
response status of 204.

### Request

#### Query Parameters
NA

| Header       | Definition |
| :---         | :--- |
| Content Type | application/json |
| Authorize    | Bearer {JWT} |
| requestID    | Unique identifier sent in the client request for logging. |

#### Body
NA

### Response

#### Http Status
| Code | Status                 | When To Use |
| :--- | :---                   | :--- | 
| 204  | No Content             | The server successfully processed the request and is not returning any content. |
| 400  | Bad Request            | The server cannot process the request (such as malformed request syntax, size too large, invalid request message framing, or deceptive request routing, invalid values in the request). For example, the API requires a numerical identifier and the client sends a text value instead, the server will return this status code. |
| 401  | Unauthorized           | The request could not be authenticated. |
| 403  | Forbidden              | The request was authenticated but it is not authorized to access the resource. |
| 405  | Not Allowed            | The method is not implemented for this resource. The response may include an Allow header containing a list of valid methods for the resource. |
| 500  | Internal Server Error  | An internal server error. |


#### Headers
| Header      | Definition |
| :---        | :---       |
| Accept Type | application/json |
| Api Version | {major}.{minor}.{patch} |
| requestId   | Unique identifier sent in the client request for loggin. |

#### Body
NA
