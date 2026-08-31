# Lab: Multi-Step Process with No Access Control on One Step


## Objective



Log in as the regular user `wiener` and exploit a flawed multi-step role-change process to promote yourself to an 
administrator.



### Credentials



```text

Administrator:

Username: administrator

Password: admin




Regular User:

Username: wiener

Password: peter

```




---




## Vulnerability




The application uses a **multi-step process** to change a user's role.





The problem is that **one of the steps does not properly enforce access control**.



Although some steps are restricted to administrators, the vulnerable step can be accessed directly by a regular user.




This results in **vertical privilege escalation**.




---



## Steps to Solve


### 1. Log In as Administrator






Log in using:


```text


Username: administrator

Password: admin

```


The purpose is to understand how the role-change process works.




---



### 2. Perform a Role Upgrade




From the Admin Panel, start the process of upgrading a user.



Monitor the requests in:


```text

Burp Suite → Proxy → HTTP history

```



The process contains multiple requests/steps.


---



### 3. Identify the Vulnerable Step



One of the requests was:



```http

POST /admin-roles HTTP/2

```



With the following body:


```text

action=upgrade&confirmed=true&username=wiener

```



This request represents the role-upgrade action.


The important issue is that this step does not properly verify whether the current user is an administrator.



---



### 4. Send the Request to Repeater



Send the request to:


```text

Burp Suite → Repeater

```



Keep the request available for testing.




---


### 5. Log In as Wiener



Log out from the administrator account and log in as:




```text

Username: wiener

Password: peter

```



---


### 6. Replace the Session Cookie



The captured request contains the administrator's session cookie.




Replace it with **Wiener's session cookie**.


The request still contains:



```text

action=upgrade&confirmed=true&username=wiener

```



but it is now sent using Wiener's authenticated session.


---



### 7. Send the Request



Send the modified request.



Because this particular step does not properly enforce authorization, the server accepts the request even though 
Wiener is not an administrator.



The result is:



```text

wiener → administrator

```



The lab is solved. ✅



---




## Why Does the Vulnerability Exist?




A multi-step process should perform authorization checks on **every sensitive step**.



A secure implementation should look like:



```text

Step 1

   ↓

Authentication + Authorization

   ↓

Step 2

   ↓

Authentication + Authorization

   ↓
Step 3
   ↓

Authentication + Authorization

```


In this lab, one step fails to perform the required authorization check:




```text

Step 1 → Authorization ✅

Step 2 → Authorization ❌

Step 3 → Authorization ✅

```



Therefore, an attacker can skip directly to the vulnerable step.



---



## Attack Flow


```text

Login as Administrator
        ↓

Perform role-change process

        ↓
Capture all requests


        ↓

Identify the unprotected step

        ↓


Send it to Repeater
        ↓


Login as Wiener

        ↓

Replace Administrator Session
with Wiener's Session
        ↓


Send the vulnerable step directly
        ↓

Wiener becomes Administrator
        ↓

Lab Solved ✅

```

---

## Important Concept




The vulnerability is **not simply because the process has multiple steps**.


The actual vulnerability is:



> **A sensitive step in the process does not enforce authorization independently.**



An attacker should never be able to rely on previous steps being authorized.



Each sensitive request must verify the current user's privileges.



---



## Lab 11 vs Lab 12



### Lab 11 — Method-Based Access Control



The issue was related to how authorization was handled based on the **HTTP method**.



```text

HTTP Method

     ↓

Flawed Access Control

     ↓

Privilege Escalation

```



### Lab 12 — Multi-Step Process



The issue is that one step in a multi-step process is not properly protected.



```text

Step 1 → Protected

Step 2 → Unprotected ❌

Step 3 → Protected

```



### Easy Way to Remember



```text
Lab 11 → HTTP Method

Lab 12 → Process Step

```



---




## Authentication vs Authorization



### Authentication


Determines **who you are**.




```text

Session Cookie

      ↓

Wiener

```


### Authorization



Determines **what you are allowed to do**.



```text

Wiener → Regular User


Administrator → Admin actions

```



In this lab, the problem is with **Authorization**.





The server knows that the request belongs to Wiener, but fails to properly verify whether Wiener is authorized to 
change the role.



---



## Key Takeaways






* Every step of a sensitive multi-step process must enforce authorization.

* Never assume that authorization in one step protects later steps.

* Test whether individual steps can be accessed directly.

* Capture and compare requests made by users with different privilege levels.

* Session cookies determine the authenticated user associated with a request.


* Authorization must always be enforced server-side.

* Sensitive actions should verify the user's privileges at the point where the action is performed.



### Vulnerability Type



**Broken Access Control**





### Impact



**Vertical Privilege Escalation**



### Related Concepts



* Broken Access Control

* Privilege Escalation

* Vertical Privilege Escalation

* Multi-Step Processes

* Session Management

* Authentication vs Authorization

* Server-Side Authorization

* Request Replay
