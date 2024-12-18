---
layout: wiki
title: AWS Red Teaming
cate1: AWS Red Teaming
cate2:
description: AWS Red Team Methodology
keywords: 
---

# Attacking AWS

## AWS Organisations
  - Management Account (Root Account) can manage policies throughout organisation and should not run any AWS services.
  - OU (Directories) can contain other OUs and accounts, the accounts of which can run AWS services.
  - The "OrganizationAccountAccessRole" is created which allows assumption of the Child account. Grants "sts:AssumeRole" privileges.
  - Management Account Policies:
    - Service Control Policies (SCPs) restrict what children account can do.
    - Limits permissions, and specifies what services and actions that users and roles in the account can do.
    - Only restricts principals in the account, but not other accounts.
    - Examples:
      - DenyStopAndTerminateWhenMFAIsNotPresent
      - organizations:LeaveOrganization

## AWS Principals
  - IAM Users --> Username + Password or API token
  - Each AWS account has a root user, which should be disactivated (via SCPs from AWS Organizations)
  - IAM MFA - can be attached to IAM user or groups, resources/services, trust policy of IAM roles - not used commonly in CLI
  - IAM User Groups - only users can be part of a group, all policies attached to groups are inherited
  - IAM Role - roles are accessible only through other principals (and require a trust policy that indicates who can access it), the role must be assumed and the princpal needs the assume role permission. Credentials are temporary.
  - IAM Boundaries & Least Privilege Principle - limits permissions that a specific user might have, only given to users. This is used in combination with SCPs to limit permissions to users.
  - IAM Policies - array of permissions given to access resources - there are AWS managed policies (easy to use), Customer Managed Policies (more granular) - inline vs attached policies. Resource policies also exist.
  - IAM Session Policies - boundaries for session credentials, limit permissions out of the existing permissions you can access, some services like Cognito use session policies to limit what the session can do.
  - IAM Identity Providers - Okta, AD, OpenID etc.
  - IAM Identity Center - Allows central management and administration of users. There can be Identity Center directory, AD or external identity provider. Must be in the management account. An IDP trusts our identity center, an SSO role is also generated and the role is trusting the identity provider. The typical use is to use active directory or github to grant identity center permissions.

Whitebox Audit: arn:aws:iam::aws:policy/ReadOnlyAccess

## Configure AWS CLI
```aws configure```
Add credentials (Access Key ID + Secret Access Key)

## Exploitation of AWS Services
### IAM Exploitation
  - Need permissions to enumerate the account. IAM:List* and IAM:Get* permissions.
  - 
    

