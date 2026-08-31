# Access Control Vulnerabilities



## Definition



Access control vulnerabilities occur when an application fails to properly enforce authorization restrictions, 
allowing users to access data or functionality they should not be allowed to access.



In simple terms:



> The application should verify **who you are** and **what you are allowed to do**. Failure to enforce these 
permissions can lead to an access control vulnerability.



---



# Types of Privilege Escalation



## Vertical Privilege Escalation



Vertical privilege escalation occurs when a user gains access to functionality or permissions belonging to a 
higher-privileged user.



Example:



```text

Normal User → Admin

```



A normal user accessing an admin-only endpoint:



```text

/admin

```



---



## Horizontal Privilege Escalation



Horizontal privilege escalation occurs when a user accesses resources belonging to another user with the same 
privilege level.



Example:



```text

User A → User B


```



Changing:


```text

/profile?id=123

```



to:



```text

/profile?id=124

```



and accessing another user's profile.



---



# Lab: Unprotected Admin Functionality



## Objective



Delete the user:



```text

carlos

```



## Steps



1. Navigate to:



```text

/robots.txt

```



2. Check the `Disallow` entries and identify the hidden admin panel path:



```text

Disallow: /administrator-panel

```



3. Navigate to:



```text

/administrator-panel

```



4. Access the admin panel.


5. Delete the user `carlos`.



---



## What is `robots.txt`?



`robots.txt` is a standard file located in the root directory of a website:


```text

https://example.com/robots.txt

```



It provides instructions to search engine crawlers about which paths should not be crawled.



Example:



```text


User-agent: *

Disallow: /admin/

Disallow: /private/

```



---


## Vulnerability



The application relied on hiding the admin panel instead of properly protecting it.




The admin panel path was disclosed through `robots.txt`, allowing an unauthorized user to access sensitive admin 
functionality.



This is an example of **Security Through Obscurity**, which is not a valid security control.



> Hiding a sensitive endpoint does not protect it. Proper authorization checks must always be enforced server-side.



---




## Vulnerability Type




* **Access Control Vulnerability**

* **Unprotected Admin Functionality**

* **Vertical Privilege Escalation**



## Key Takeaway



> Never rely on hidden URLs to protect sensitive functionality. Always enforce authorization checks on the server 
side.

