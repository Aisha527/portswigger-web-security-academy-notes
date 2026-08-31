# Lab: Method-Based Access Control Can Be Circumvented



## Objective



Log in as the regular user `wiener` and exploit the flawed access control to promote the account to an administrator.




### Credentials



```text id="zq7x2p"

Administrator:

Username: administrator

Password: admin



Regular User:

Username: wiener

Password: peter

```



---




## Vulnerability



The application implements access controls that depend partly on the **HTTP method** used by a request.



The authorization checks are flawed, allowing a regular user to reuse an administrative request and execute a 
privileged action.



---



## Steps to Solve



### 1. Log In as Administrator



Log in using:



```text id="k3v8qa"

Username: administrator

Password: admin

```



Access the Admin Panel and locate the functionality used to promote a user.



---



### 2. Capture the Upgrade Request




Perform the user upgrade action and inspect the request in:



```text id="n5c2wd"

Burp Suite → Proxy → HTTP history

```



Pay attention to:




```text id="x7m1zp"

HTTP Method

URL

Parameters

Session Cookie

```



Send the request to **Repeater**.



---



### 3. Log In as Wiener



Log out from the administrator account and log in using:



```text id="q6r4mb"

Username: wiener

Password: peter

```



---



### 4. Replace the Session Cookie



The captured upgrade request contains the administrator's session cookie.



Replace the administrator's session cookie with **Wiener's session cookie**.



The request is now sent using Wiener's authenticated session.



```text id="a8y3kc"

Administrator Request

        ↓

Copy the upgrade request

        ↓

Replace Administrator Session Cookie

        ↓

Wiener's Session Cookie

        ↓

Send Request

```



Because of the flawed authorization checks, the server accepts the privileged action.



---



### 5. Verify the Privilege Escalation



Wiener's account is successfully promoted to:



```text id="d5w9vn"

Administrator


```




The lab is solved. ✅



---



## Why Does the Vulnerability Exist?




The application does not properly verify whether the **currently authenticated user** is authorized to perform the 
administrative action.



A secure application should perform authorization checks for every privileged request:



```text id="j7p2xs"

Request

   ↓

Identify the user


   ↓

Check user's privileges

   ↓

Is the user an administrator?

   ↓

Yes → Allow

No  → Deny

```



Instead, the application has flawed method-based access control, allowing the privileged request to be reused with a 

regular user's session.



---


## Authentication vs Authorization



### Authentication


Determines **who the user is**.



For example:



```text id="m3x7qa"

Session Cookie

      ↓

Wiener

```



### Authorization



Determines **what the user is allowed to do**.



For example:



```text id="f8v2nc"
Wiener → Regular User


Administrator → Admin actions

```



In this lab, the problem is with **Authorization**, not Authentication.




The server correctly identifies Wiener, but incorrectly allows him to perform an administrator-only action.



---



## Role of the Session Cookie



The session cookie identifies the authenticated session.



For example:



```text id="v6k1wp"

Administrator Session Cookie

          ↓

Administrator

```



After replacing it:



```text id="r4m8zy"

Wiener's Session Cookie

          ↓

Wiener

```



The same administrative request is now sent as Wiener.



This is useful when testing whether the server actually performs proper server-side authorization.


---



## Attack Flow



```text id="b2x7mq"

Login as Administrator
        ↓

Perform privileged action

        ↓

Capture the request
        ↓

Send to Repeater

        ↓


Login as Wiener

        ↓

Replace Administrator Cookie
with Wiener's Cookie

        ↓

Send the request

        ↓

Wiener becomes Administrator

        ↓

Lab Solved ✅

```



---



## Key Takeaways



* Authentication and authorization are different concepts.


* A valid session does not automatically mean the user is authorized for every action.

* Every privileged action must have a server-side authorization check.
* HTTP methods should not be relied upon as the primary authorization mechanism.

* Session cookies identify the current authenticated user.

* When testing access control, compare requests made by users with different privilege levels.

* Replaying an administrative request with a low-privileged session is a useful access control test.

* Privileged actions should be explicitly authorized for the current user.



### Vulnerability Type



**Method-Based Access Control Bypass**



### Impact


**Vertical Privilege Escalation**



### Related Concepts



* Broken Access Control

* Privilege Escalation

* Vertical Privilege Escalation

* HTTP Methods

* Session Management

* Authentication vs Authorization


* Server-Side Authorization
