---
layout: wiki
title: Scoping Requirements
cate1: Scoping
cate2:
description: Pre-engagement questions for technical scoping
keywords: Scoping
---

# Scoping Requirements

## Scoping Overview

The objective of the scoping meeting is to gain an understanding of what is to be tested, why the test is being performed, and any particular risks and concerns that the team has.

During this meeting, the security team will work through a list of primarily technical questions to gain an in-depth understanding around the functionality of the systems being tested.

This is also an opportunity for the team to gain an understanding of how the testing will be performed and raise any concerns they may have. 
To maximize the value of the assessment, it would be great if we can get a walkthrough of any in-scope systems.

## Administration

### Point of Contacts

- **Project Manager:** [Project Manager]
- **Technical Contact:** [Developer or Manager]

## Purpose of Meeting

### Opening Statement

The purpose of this meeting is to scope the upcoming penetration test for the [application_name] application. This is to gain an understanding of the functionalities of the systems to be tested, and also provide an opportunity for the developers to raise any concerns regarding the test.

### Questions to Ask

- What is the reason/purpose for this assessment?
- Has the team completed a risk assessment/threat model prior to this meeting?
- Have there been any previous penetration tests?
- Are there any previous vulnerability reports that can be provided?

## Introduction

- Is it possible for a brief description/summary of the application?
- Can you please walk us through the application?

## Scope

### Components in Scope

Which components are considered in-scope that the team wants us to test? These may include:

- Internet-facing web application - Web application
- Administrative portals - Web application
- Range of IP addresses/DNS Hostnames - External infrastructure
- Source Code/Github/Bitbucket (read-only) of application - Source code review
- AWS/Azure (read-only) - Cloud configuration review

### Other Items in Scope

- Is there anything else that should be in scope?
- Is there anything else you are worried about that may be internet-facing? - External infrastructure
- Are there any mission-critical systems where extra care should be taken during testing?

These may include:

- Mission-critical systems
- Other internet-facing devices

### Environment

- What environments will be used for testing?
- What are the differences between these environments and production?

### Threats and Risks

- Do you have any areas of concern?
- What is the classification of user data?

### Documentation

- Do you have API documentation and test data? - API testing
- Do you have architecture diagrams?
- Are there any screenshots/documentations of important flows?

### Timing Estimations

- How many hosts are to be scanned? - External infrastructure
- How many hosts are to be penetration tested? - External infrastructure
- Is authentication required? - External infrastructure
- How large is the application approximately? - Web application
  - e.g. how many screens/pages are there (10, 50, 100, …)
  - e.g. dynamic pages vs static pages
- How many user roles are to be tested? - Web application
- How many API endpoints are there? - API testing
- Penetration testing or vulnerability scanned (inc. Databases) - Web application
- How many access points are to be tested? - Wireless
- How many floors are to be tested? - Wireless
- Any areas of interest you would like tested in particular? - Wireless

### Future Proofing

- Do you have a roadmap or list of any newly planned functionality that is going to be deployed in the near future?

## Testing Requirements

- Can you provide us with test accounts prior to the commencement of the testing date?
  - At least two per account type or role
  - Up to three account types (low level, high level, admin)
  - Accounts should be split between two different companies or customers

- Do we need permission to test?
- Do any third parties need to be notified?
- Are there any time restrictions on testing? The majority of testing activities will occur between 0830 and 1730 Australian time, however testing is not restricted to these hours unless specified.
- Do we need to whitelist IP addresses?
  - What IP addresses will the test traffic originate from?

## Reporting Requirements

- What are the reporting requirements?
  - Formal report with risk ratings
  - Summary of findings via email
  - Walkthrough meeting to discuss identified issues
  - Spreadsheet only report

- Do you have your own risk matrix?
- Are there any deadlines that need to be taken into consideration?
- Is there a go-live date that the developers have in mind?

## Consulting Specific

- Desk space for the duration of testing
- Network access for testing laptop
- Access pass for the office if required
- Do we need to send daily testing notifications? If so, who needs to receive these?

---

## Resources

- [PentesterLab Scoping](https://blog.pentesterlab.com/scoping-f3547525f9df)
