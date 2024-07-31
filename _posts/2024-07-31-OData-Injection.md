---
layout: post
title: OData Injection
categories: [OData, Injection]
description: Introduction and attacks against OData API
keywords: OData, Injection
mermaid: false
sequence: false
flow: false
mathjax: false
mindmap: false
mindmap2: false
---

# OData Injection 
OData (Open Data Protocol) according to [Microsoft](https://learn.microsoft.com/en-us/odata/overview) is an ISO/IEC approved, OASIS standard that defines a set of best practices for building and consuming REST APIs. The intention for OData is to simplify the querying and sharing of data across different systems by providing a uniform way to structure and interact with the data.

## OData Query

A complete query for OData looks like this (credits to [prospect365](https://docs.prospect365.com/en/articles/2455275-getting-started-with-the-odata-api):

```
https://odata-v1.example.com/Contacts
    ?$select=Title,Forename,Surname,Email,StatusFlag
    &$filter=contains(email,'@example.com') and StatusFlag eq 'A'
    &$expand=Leads(
        $select=Description,Created,StatusFlag;
        $filter=StatusFlag eq 'A' and Status/DeadFlag eq 0;
        $expand=Status(
            $select=Description
        )
    )
```
### Base URL
```
https://odata-v1.example.com/
```
This is the API server location where the request will go to. The service is usually protected by bearer token authentication, so if a user has access to a valid token they may be able to make calls to the OData service.

### Entity
```
/Contacts
```
This is the CRM record that the user wishes to access. This will attempt to retrieve everything from within the records, which is effectively like retrieving everything from an SQL table. This can take a very long time, and usually OData would segment the data retrieval to a next link.

### Parameters and Function
In order to perform a meaningful search, OData allows the use of specific functions (system queries) that are used to narrow down the results. These functions are similar to SQL syntax. These are:

  - **$select**: retrieve only a particular field from the search query
  - **$expand**: in OData there are "navigation properties" where one entity references/links to another - if the user wants to retrieve the links of a particular entity as well, they need to use the "expand" function
  - **$filter**: is used to reduce the dataset by choosing a field and determining the requirement
  - **$orderBy**: exactly as it sounds, this function re-arranges and sorts the data in a particular way before returning it to the user, either ascending or descending
  - **$top**: limits the number of records returned
  - **$skip**: skips the number of records specified

## Attacking OData
The typical flow to interact with the OData API is as follows:
  - The OData API receives a HTTP request from the client. Typically we could be testing the OData API directly on a client API server (such as https://odata-v1.example.com/), but there are also circumstances where a web application would query the OData service, apply their own limitations to the use of the API.
  - The OData API would parse the request, taking into consideration the various parameters and functions that are supplied to it.
  - The OData API will construct an appropriate request to the data source, for example an SQL query if the request is to be handled by an SQL database. Thus, SQL injection techniques are viable if the OData API is not properly secured.
  - The data is processed and a response is provided to the client.

An OData injection vulnerability allows attackers to manipulate OData queries to disclose sensitive information from the underlying data source. The data source could vary, and could be either relational or non-relational databases, in-memory data, file systems or external services. By exploiting this vulnerability, an attacker can craft malicious OData queries to retrieve confidential data and other sensitive information that should not be exposed.

Provided below is a general checklist an auditor should look out for when attacking the OData API:
  - **Access the service (metadata) document**
    - This is usually a JSON or XML document located at the top-level domain that provides metadata on all the entities, function imports and other resources available through the service. We can leverage this to construct our own URLs to query and determine if there is sensitive data located in the endpoints that we discover.
  - **Atypical Web API testing**
    - This includes testing for insecure HTTP methods, user privilege and access control, authentication and authorization, logic flaws, data exposure etc.
  - **Data Validation and Error Handling**
  - **OData Injection into System Queries (parameters and functions)**

### OData Injection into System Queries

In OData, using the wildcard character in different functions can expose an additional subset of data, which may not be intended by the creator.

For example:

```
$select=*
```

Can be used to retrieve additional hidden properties that contain sensitive information. This can include additional metadata that reveals the filters, query type and other information associated with the query. Alternatively, we can try to create a TRUE condition in the filter to ensure that all the data is returned to the attacker, as shown below:

<img width="653" alt="image" src="https://github.com/user-attachments/assets/5a2d9fa9-a0cf-4c0f-bba2-5bbb2471faa1">


```
Filter=true)+or+true(
$filter=true) or true( // equivalent
```

This ensures that the filter only produces a TRUE condition, revealing additional information about other users that should not be present in the response. The redacted filter is provided below:

```
"filter":"([Insert_User_Filter_Expression_Here]) and (AllowClaims/any(c: search.in(...) and LocationId eq '...')"
"filter":"(true) or (true) and (AllowClaims/any(c: search.in(...) and LocationId eq '...')" // overrides other filters
```

While this is one example, it is sufficient to demonstrate that injection vulnerabilities can be used to bypass OData filters.
