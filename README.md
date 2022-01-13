<style>
  H1{color:Blue !important;}
  H2{color:DarkOrange !important;}
</style>

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

- Australian Government - API Design Standard
- White House Web API Standards
- JSON Schema Specification
- OData 4.01
- Standards for Event Driven APIs
- Refinery API Standards
- Open API Guide

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
will be the name of the resource w/ ‘_id’ appended to the end, e.g. account_id, 
user_id when using the snake_case naming convention. The following are examples 
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
