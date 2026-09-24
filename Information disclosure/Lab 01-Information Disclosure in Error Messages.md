# Information Disclosure in Error Messages



**Difficulty:** Apprentice

**Category:** Information Disclosure / Information Leakage

**Status:** Solved ✅



## Definition — Information Disclosure






**Information Disclosure** is a vulnerability that occurs when an application unintentionally exposes sensitive or technical information 
that should not be available to users.



Examples of potentially disclosed information include:



* Usernames or user data

* Source code

* API keys and credentials

* Database information

* Internal file paths

* Framework names and versions

* Server or programming language information

* Detailed error messages and stack traces



Technical information may not always have a direct impact, but it can help an attacker understand the application's technologies and attack 
surface and potentially identify other vulnerabilities.



---


## 🎯 Lab Objective



The application uses **verbose error messages** that reveal information about the third-party framework being used.




The goal is to obtain and submit the **framework version number**.



---



## 🧠 Vulnerability Concept



In this lab, the information disclosure occurs through a **verbose error message**.



Instead of returning a generic error, the application exposes details about the exception and the framework.




---



## 🔎 Step 1 — Identify an Input Parameter



The application contains a `productId` parameter that expects an integer:





```http

GET /product?productId=1

```



---



## 💥 Step 2 — Trigger an Error



We supplied an invalid value:



```http

GET /product?productId=' HTTP/2

Host: <LAB-HOST>

```



---



## ⚠️ Step 3 — Analyze the Error



The application returned:



```text

HTTP/2 500 Internal Server Error

```



The response contained:



```text

java.lang.NumberFormatException: For input string: "'"

```

This indicates that the backend attempted to convert the supplied value into an integer.



Conceptually:



```java

Integer.parseInt("'")

```



This resulted in:




```text

NumberFormatException

```



---


## 🔓 Step 4 — Extract the Disclosed Information



The response also revealed:



```text

Apache Struts 2 2.3.31

```




Therefore:


```text

Framework: Apache Struts 2

Version: 2.3.31

```




---



## 🎯 Step 5 — Submit the Version




The required answer was:



```text

2.3.31


```


Submitting it solved the lab. ✅



---



## 💡 Key Technique



```text

Find an input parameter



        ↓


Send unexpected input

        ↓

Trigger an application error

        ↓

Inspect the verbose response

        ↓

Identify leaked information

        ↓

Extract the framework/version

```



---



## 🔑 Key Takeaway




Information disclosure is not limited to passwords or API keys.



Technical information such as:




```text

Framework

Framework version

Java version

Server version

File paths

Stack traces

```



can help an attacker understand the application's attack surface.



The important consideration is the **impact and exploitability** of the disclosed information.
