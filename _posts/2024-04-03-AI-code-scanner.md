---
layout: post
title: Finding Security Vulnerabilities using an AI code scanner
categories: [AI, Tools]
description: AI code scanner
keywords: Code Review, AI, Tools
mermaid: false
sequence: false
flow: false
mathjax: false
mindmap: false
mindmap2: false
---

# Introduction
Secure Code Review is an integral part of the software development life cycle, and is also a crucial part of white-box penetration testing. Numerous methods exist for scanning source code for security vulnerabilities, from using tools like Graudit or Semgrep to detect known vulnerability signatures, to leveraging SonarQube or Snyk for taint analysis and dependency checks, or employing semantic analysis tools such as CodeQL. Each approach offers its unique advantages.

In this article, we aim to explore another potential tool that can be used to perform secure code review - by employing large language models (LLMs) as an innovative alternative for identifying security vulnerabilities during an engagement.

# The Issues
There are some potential issues that should be taken into account when considering an AI code scanner. The most obvious is privacy. When performing a consulting engagement or internal penetration test, it is crucial to avoid sending proprietary source code to third parties such as OpenAI. While industry leaders are increasingly prioritising organisational privacy, with offerings like ChatGPT for Enterprise as an example, maintaining a cautious approach with sensitive code remains essential.

As such, we aim to use open source models that are available through [Hugging Face](https://huggingface.co/), and use LM Studio to setup a local instance for our script to communicate with.

Another issue is reliability. AI models have a tendency of hallucinating, and can provide false positives by creating vulnerable code that does not exist, or false negatives by failing to identify (sometimes obvious) security issues. The models that we use in this experiment are general purpose and not specifically designed for code scanning. Therefore the use of AI code scanning should only form a piece of a greater puzzle, and other tools and manual review is still required.

# Setup
The setup is simple to create a basic AI code scanner. We can use [LM Studio](https://lmstudio.ai/) to download compatible LLMs to run locally. This will also mean that we are able to scan code on our machine even while offline.

<a href="/images/blog/ai-scanner-1"><img src="/images/blog/ai-scanner-1.png"></a>
<p align="center">Searching for a compatible LLM via LM Studio</p>

LM Studio provides the functionality to search and download compatible LLMs within the UI, so there is no need to download models via Hugging Face. The model that is chosen should be based on the capabilities of the machine taking into consideration the GPU usage and other factors. In this circumstance we are using the Mixtral-8x7B-Instruct-v0.1-GGUF.

<a href="/images/blog/ai-scanner-2"><img src="/images/blog/ai-scanner-2.png"></a>
<p align="center">Starting the Local Inference Server on localhost:1234</p>

We can then start the local inference server by loading the model. The default location should be accessible at localhost:1234.

# Scanner Code
We provide an AI code scanner code in the following location: [AI-Code-Scanner](https://github.com/qwutony/AI-Code-Scanner/). Follow the instructions to install the necessary requirements to run the Python script.

```bash
git clone https://github.com/qwutony/AI-Code-Scanner.git
cd AI-Code-Scanner
pip install -r requirements.txt
python scanner.py [Code Repository] --file-types [File Extensions] (--scan-all)
If you want to install Python dependencies in an isolated Python environment, using virtualenv instead.
To demonstrate the capabilities of this tool, we used the code from several vulnerable code snippet repositories. These include:
https://github.com/yeswehack/vulnerable-code-snippets
https://github.com/snoopysecurity/Vulnerable-Code-Snippets
```

The output of the scan looks like this:

<a href="/images/blog/ai-scanner-3"><img src="/images/blog/ai-scanner-3.png"></a>
<p align="center">Potential Command Injection in a PHP file</p>

If the LLM identifies a potential security vulnerability, it would print the location of the vulnerable file in the scanned directory, the line number (although this is inconsistent since it originates from the LLM), the type of vulnerability, and the code snippet of the vulnerable code.

<a href="/images/blog/ai-scanner-4"><img src="/images/blog/ai-scanner-4.png"></a>

Another example is shown here, where the LLM successfully identifies the XXE vulnerability in the PHP file.

<a href="/images/blog/ai-scanner-5"><img src="/images/blog/ai-scanner-5.png"></a>
<p align="center">Another screenshot identifying command injection vulnerabilities.</p>

However, there are circumstances where the application fails to identify the vulnerability, or hallucinates and "fabricates" an entirely new vulnerability despite it not existing. Therefore, the use of LLMs are not entirely reliable and should only be used in combination of manual reviews.

This is the current prompt that is used for the script - if the LLM returns FALSE then no security vulnerability is identified, otherwise the response will be printed out into the console.

> Scan the provided code chunk for security vulnerabilities - there is a high chance there are no vulnerabilities in the code. You MUST separate each security issue identified with the characters @@@@. Only respond with 'FALSE' if there are no vulnerabilities. If a security vulnerability is present, respond with 'TRUE' then the line numbers where the issue is identified, followed by the issue type in concise wording, and then finally the vulnerable code snippet but very concise. The response should be in the following format: TRUE | line_number(s) | issue_type | code_snippet."}

# Conclusion
As LLMs continue to evolve and improve, there is greater opportunity to leverage these models to perform code scanning for security auditing purposes. While it is still in its infancy, there is potential that this form of code scanning could be useful for penetration testers and auditers in the future.
