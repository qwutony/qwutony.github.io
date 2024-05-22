---
layout: post
title: "Customising SQLMap: Integrating Personalised Injection Payloads"
categories: [SQLi]
description: some word here
keywords: SQL Injection, SQLMap, Inference
mermaid: false
sequence: false
flow: false
mathjax: false
mindmap: false
mindmap2: false
---

# Introduction
At times, SQLMap might mistakenly assess a parameter as secure due to various factors, despite evidence from manual testing indicating otherwise.

We are familiar with the standard flag modifications that SQLMap provides to scan for vulnerabilities - level and risk. Increasing level increases the effective scope for testing with more payloads and boundaries, while increasing the risk allows for heavy time-based injections and OR-based SQL injections.

We can also use SQLMap tamper scripts to obfuscate the payload to bypass filters and WAFs, but there are times where these techniques do not provide complete coverage and identify the vulnerability.

In that case, we can modify the payload XML files that SQLMap stores in their libexec directory and specify the exact payload that is required by adding an additional test case to the file. This was initially discussed in an SQLMap issue located here: https://github.com/sqlmapproject/sqlmap/issues/4091.

**MacOS**
For MacOS users that installed SQLMap via brew we can navigate to the following location to find the XML payload files:
```
    /opt/homebrew/Cellar/sqlmap/[version]/libexec/data/xml/payloads
```
**Linux**
For Linux users the XML payload files should be located here:
```
    /usr/share/sqlmap/data/xml/payloads
```

# Recent Example
The following is a recent example of an XOR boolean-based and time-based blind SQL injection that was manually discovered, however SQLMap was unable to determine the injection location without modification of the XML payload files. There were several issues which will be outlined in the subsequent sections.

## Initial Attack Vector

The initial discovery of the SQL injection was relatively straightforward - a POST request contained the searchline parameter which produced a different status code (500) when double quotes were provided in the search query. Further investigation determined that the application was susceptible to time-based SQL injection, and the following payload could be used to conjure TRUE - FALSE conditions to enumerate the database:

```
"XOR(if(now()=sysdate(),SLEEP((0.25)),0))XOR" // There was a significant delay by a factor of 10, so sleep(0.25) would result in a 2.5 second time delay
```

## Inference-based SQL injection
Inference-based SQL injection take a significant time to extract sensitive information from the database, especially via time-based SQL injection. Fortunately, we were able to determine that the application was also vulnerable to boolean-based SQL injection, which provided a slightly faster avenue to extract data:

```
"XOR(if((select/**/666/**/where/**/1=1),444,0))XOR" // This would result in a TRUE condition returning a response of ~21,000 bytes
"XOR(if((select/**/666/**/where/**/1=2),444,0))XOR" // This would result in a FALSE condition returning ~27,000 bytes
```

The Burp request(s) are as follows (only the POST request is shown and mostly redacted):
<a href="/images/blog/sql-burp-1.png"><img src="/images/blog/sql-burp-1.png"></a>
<p align="center">TRUE condition with ~21,000 byte response</p>

<a href="/images/blog/sql-burp-2.png"><img src="/images/blog/sql-burp-2.png"></a>
<p align="center">FALSE condition with ~27,000 byte response</p>

We can employ the use of SQLMap for automated enumeration of the database, however SQLMap failed to detect the vulnerability:
Initial SQLMap query fails to locate the vulnerabilityThe following flags were provided for testing:

```
-u [location] OR -r [request]
 --data=[POST data]
-p "searchline"
 --random-agent [WAF evasion]
 --dbms=mysql
 --level 1 --risk 1 (as we know the exact location + injection vulnerability, but alternated through different levels/risks)
 --technique=B (Boolean-based blind SQL Injection)
 --tamper=between,space2comment (and others)
 --headers ...
 --prefix="sqli" --suffix="" (prevent SQLMap from adding custom prefix/suffixes)
```

## Initial Problems
There were several factors that made enumeration difficult however:

  + Time-based SQL Injection was cumbersome, and SQLMap requires a minimum sleep time of 1 second. This sleep value is amplified by a factor of 10 on the application, which means sleep(1) would result in a 10 second delay. This would often miss the timing for SQLMap to receive a valid response. This was circumvented by using Boolean-based SQL Injection payloads instead.

  + A Web Application Firewall (WAF) blocked many SQL sensitive strings, including information_schema.tables, which meant table and column names are required to be brute-forced. An eventual WAF bypass was discovered, however this is beyond the topic of this article.

  + SQLMap does not have the XOR payload by default, and so we need to manually add in the test vector.

## Creating New Test Cases
As previously mentioned, we can modify the XML payload files to provide the exact test cases in the event that SQLMap does not recognise our attack vector. These files are located in the /data/xml/payload SQLMap data files depending on where it was installed. We can specify the test element as follows:

```xml
    <test>
        <title>MySQL XOR boolean-based blind</title>
        <stype>1</stype>
        <level>1</level>
        <risk>1</risk>
        <clause>1-8</clause>
        <where>1</where>
        <vector>"XOR(if((select/**/666/**/where/**/[INFERENCE]),444,0))XOR"</vector>
        <request>
            <payload>"XOR(if((select/**/666/**/where/**/[RANDNUM]=[RANDNUM]),444,0))XOR"</payload>
        </request>
        <response>
            <comparison>"XOR(if((select/**/666/**/where/**/[RANDNUM]=[RANDNUM1]),444,0))XOR"</comparison>
        </response>
        <details>
            <dbms>MySQL</dbms>
        </details>
    </test>
```

The main elements we are concerned about are vector, request and response. We can specify our attack payload in the vector element, and replace the inference logic with [INFERENCE] to point SQLMap to the injection location. The request and response elements accepts the same payload, but replaces the inference with random numbers [RANDNUM]. We should specify a TRUE condition for the request, and a FALSE condition for the response (notice the [RANDNUM1] to indicate a different random number).

## Successful Exploitation
The newly added inference logic allowed SQLMap to create a TRUE/FALSE condition based on the response length, however it was still unable to identify the vulnerability. To circumvent this, we can use the string feature to match a string in the response that is unique to one condition, such as a PNG file. There are other detection methods provided in SQLMap which I encourage the readers to familiarise with:
```
--string=STRING     String to match when query is evaluated to True
--not-string=NOT..  String to match when query is evaluated to False
--regexp=REGEXP     Regexp to match when query is evaluated to True
--code=CODE         HTTP code to match when query is evaluated to True
```
# Conclusion
By providing the previous flags, adding a new test case and providing a string match condition, we were able to enable SQLMap to correctly identify the XOR boolean-based blind vulnerability, and enumerate the database successfully.
