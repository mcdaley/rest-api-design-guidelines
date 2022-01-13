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

```
GET /accounts/123456
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
```

This representation can change over time as the system and data within 
changes. A future call to this same endpoint may yield a different 
representation if the account balance changes or if the account is blocked.

```
GET /accounts/123456
200 OK
Content-Type: application/json

{
  “Id”:                “123456”,
  “nickname”:          “Spending”,
  “routing_number”:    “123456789”,
  “available_balance”: 400.00,
  “posted_balance”:    400.00,
  “status”:            “blocked”
}
```

### TEST trying to make a section of code bold
<pre>
  <b><i>GET /accounts/123456</i></b>
  200 OK
  Content-Type: application/json

  {
    “Id”:                “123456”,
    “nickname”:          “Spending”,
    “routing_number”:    “123456789”,
    <span style="color:#ff6347">
    “available_balance”: 400.00,
    “posted_balance”:    400.00,
    “status”:            “blocked”
    </span>
  }
</pre>

## Namespace
A namespace is an optional way for a service to define a grouping of 
related functions within the system. Namespaces could be high-level 
such as a cobrand-1 or low-level such as loans or accounts.
