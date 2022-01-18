# Rest API Design Guidelines

The following document provides detailed guidelines and examples for designing 
RESTful Application Programming Interfaces (APIs) that follow the OPEN API 
specification. The goal of the document is to define a set of standards and 
best practices for building APIs, so that product, engineering, and QA teams 
can focus on the business logic of the APIs.

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

For example, in a banking application some of the resources are **customers**, 
**accounts**, **loans**, and **transactions**.

## Resource Identifier
Every resource that is available within a system (e.g. every customer, account, 
and transaction) must be uniquely identifiable within the system. This is a 
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
account information you will receive a representation of the account 
as shown below:

<pre>
  <b>GET /accounts/123456</b>
  200 OK
  Content-Type: application/json

  {
    “id”:                “123456”,
    “nickname”:          “Checking”,
    “routing_number”:    “123456789”,
    “available_balance”: 1000.00,
    “posted_balance”:    900.00,
    “status”:            “active”
  }
</pre>

This representation can change over time as the system and data within 
changes. A future call to this same endpoint may yield a different 
representation if the account balance changes or if the account is blocked.

<pre>
  <b>GET /accounts/123456</b>
  200 OK
  Content-Type: application/json

  {
    “id”:                “123456”,
    “nickname”:          “Spending”,
    “routing_number”:    “123456789”,
    <b><i>“available_balance”: 400.00,</b></i>
    <b><i>“posted_balance”:    400.00,</b></i>
    <b><i>“status”:            “blocked”</b></i>
  }
</pre>

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
GET /api/v1/accounts
GET /api/v1/team-members
```

**Note:** This is the only place where hyphens are used as a word separator. 
In nearly all other situations camelCase OR snake_case, ( _ ), MUST be used.

#### Resource Identifier
When performing an action on a specific resource then the Id in the documentation 
will be the name of the resource w/ ‘_id’ appended to the end, e.g. **account_id**, 
**user_id** when using the snake_case naming convention. The following are examples 
of naming Ids:

```
GET /api/v1/accounts/{account_id}
GET /api/v1/accounts/{account_id}/transactions/{transaction_id}
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
nest the resources in the URI, for example to get the transactions associated 
with an account we can have:

```
// Return all transactions for an account
GET /accounts/{account_id}/transactions

// Return a single transaction for an account
GET /accounts/{account_id}/transactions/{transaction_id}
```

Using the functional hierarchy is recommended over adding query parameters, 
for example to get all of an account’s transactions do NOT make the URI: 

```
GET /transactions?account_id=1234. 
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
of 1001 or 1002.

## Addresses
An address is a system resource and will only support US addresses for the 
initial product launch. The following format is proposed based on the 
lob.com APIs which is a direct mail service and has the following properties:

| Field           | Required | Format                         | Description |
| :---            | :---     | :---                           | :--- |
| name            | Yes      | <= 40 characters               |
| company         | No       | <= 40 characters               | Optional company name for the user. |
| address_line1   | Yes      | <= 64 characters               | The primary number, street name, and directional information. |
| address_line2   | No       | <= 64 characters               | An optional field containing any information which can't fit into line 1. |
| address_city    | Yes      | <= 200 characters              | |
| address_state   | Yes      | 2 letter state short-name code | Enumerated list of 50 states, e.g. CA, NY, ... |
| address_zip     | Yes      | ^\d{5}(-\d{4})?$               | Must follow the ZIP format of 12345 or ZIP+4 format of 12345-1234. |
| address_country | Yes      | 3 letter ISO country code.     | Enumerated list of country codes, e.g., USA, CAN |

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
GET /customers
GET /accounts
GET /accounts/456789
GET /accounts/456789/transactions
GET /accounts/456789/transactions/123
```

### POST
Creates a new resource.

```
POST /customers
POST /accounts
POST /accounts/456789/transactions
```

### PUT
Updates all of the fields in a resource.

```
PUT /customers/john-doe
PUT /accounts/456789
PUT /accounts/456789/transactions/123
```

### PATCH
Update some of the fields in a resource.

```
PATCH /customers/john-doe
PATCH /accounts/456789
PATCH /accounts/456789/transactions/123
```

### DELETE
Deletes a resource.

```
DELETE /customers/john-doe
DELETE /account/456789
DELETE /accounts/456789/transactions/123
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
to reduce the amount of information that is sent with each response. The following 
query parameters are used to support pagination.

### page
The page that the user wants to return, for example:

```
page=1
```

Where the default page is 1.

### page_size
The number of results per page the user wants to return, for example:

```
page_size=10
```

Where the default page_size is 25 and the maximum is 100.

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
the possible response codes for each type of request are defined in the  
REST API Standard Request and Responses section of the document.

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
    “total_count”: 100,
    “page”:        2,
    “page_size”:   25
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
    “total_count”: 100,
    “page”:        2,
    “page_size”:   25
  },
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
| id              | Optional  | Unique identifier for the specific error that can be searched for in the application log files. |
| httpStatus      | Optional  | HTTP Status of the response. |
| code            | Mandatory | An application specific numeric error code |
| name            | Mandatory | The error’s name |
| resource        | Mandatory | The resource where the error occurred. |
| message         | Mandatory | Text that specifies the error, for example **Account Id “456789” is not found**. |
| stack           | Optional  | A stack trace of the error that can be used in development and testing. |

Below is an example error message:

```
{
  “errors”: [{
    “httpStatus”: 404,
    “id”:         “86032cbe-a804-4c3b-86ce-ec3041e3effc”,
    “code”:       1101,
    “resource”:   “Accounts”,
    “name”:       “Resource Not Found”,
    “message”:    “Account ID “xys786” is not found”
  }]
}
```

# Event Notification
TBD

# Security
TBD

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



















