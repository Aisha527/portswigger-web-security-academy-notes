# Lab: User ID Controlled by Request Parameter


## Objective



Obtain the API key of the user `carlos` and submit it as the solution.



### Credentials



```text id="f3x7qa"

Username: wiener

Password: peter

```



---



## Vulnerability



The application contains a **horizontal privilege escalation** vulnerability on the user account page.



The application uses a user-controlled **User ID** from the request to determine which account information to 
display, without properly checking whether the current user is authorized to access that account.







---



## Steps to Solve




### 1. Log In


Log in using:




```text id="j8m2vp"

wiener:peter

```



---



### 2. Access the Account Page



Open the **My Account** page and inspect the request using Burp Suite.



The request contains a user identifier:



```http id="w6k4zn"

GET /my-account?id=wiener

```



The important part is:




```text id="p3r9tc"

id=wiener

```



---



### 3. Modify the User ID



Change:



```text id="q7v2mx"

id=wiener

```



to:



```text id="a5k8wd"

id=carlos

```


Send the modified request:



```http id="n4y6pz"

GET /my-account?id=carlos

```



---




### 4. Access Carlos's Account



The application returns Carlos's account page without checking whether `wiener` is authorized to access it.



---



### 5. Obtain the API Key



Find the **API key** displayed on Carlos's account page and submit it as the lab solution.



The lab is solved. ✅



---



## Why Does the Vulnerability Exist?



The application trusts a **user-controlled identifier** to determine which user's data should be returned.



```text id="r6c2yk"

GET /my-account?id=wiener

          ↓

      Change ID

          ↓

GET /my-account?id=carlos

          ↓

Carlos's account

```



The server fails to verify that the authenticated user is authorized to access the requested account.




---



## Horizontal Privilege Escalation



This is an example of **Horizontal Privilege Escalation**:



```text id="x3m7qa"

User A (wiener)

       ↓

User B (carlos)

```



Both users have the same privilege level, but `wiener` can access `carlos`'s data.



### Horizontal vs Vertical



```text id="n8p4vz"

Horizontal:

User A → User B

Same privilege level



Vertical:

Normal User → Admin

Higher privilege level

```



---



## IDOR



This vulnerability is also related to **IDOR (Insecure Direct Object Reference)**.


IDOR occurs when an application uses a user-controlled identifier to access an object without properly checking 
authorization.



Common examples include:



```text id="k5s9wx"

User ID

Account ID

Order ID

File ID

```



---



## Key Takeaways



* Never trust user-controlled IDs for authorization.

* Changing an identifier should not grant access to another user's data.

* Always perform authorization checks server-side.

* Test parameters such as `id`, `user_id`, `account_id`, and similar identifiers.

* Accessing another user's data at the same privilege level is **Horizontal Privilege Escalation**.



### Vulnerability Type



**User ID Controlled by Request Parameter**



### Related Concepts



* Access Control

* Broken Access Control

* Horizontal Privilege Escalation

* IDOR

* Parameter Tampering

* Server-Side Authorization

