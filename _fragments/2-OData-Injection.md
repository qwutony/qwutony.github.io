---
layout: fragment
title: OData Injection
tags: [OData, Injection, REST]
description: Exploiting OData Injection Vulnerabilities to expose sensitive RESTful API information
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
https://crm-odata-v1.example.com/Contacts
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
https://crm-odata-v1.example.com/
```
This is the API server location where the request will go to. The service is usually protected by bearer token authentication, so if a user has access to a valid token they may be able to make calls to the OData service.

### Entity
```
/Contacts
```
This is the CRM record that the user wishes to access. This will attempt to retrieve everything from within the records, which is effectively like retrieving everything from an SQL table. This can take a very long time, and usually OData would segment the data retrieval to a next link.

