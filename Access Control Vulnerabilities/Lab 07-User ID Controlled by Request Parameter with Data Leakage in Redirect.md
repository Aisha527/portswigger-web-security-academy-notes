# Lab: User ID Controlled by Request Parameter with Data Leakage in Redirect


## Objective



Obtain the API key of the user `carlos` and submit it as the solution.



### Credentials



```text id="u4q2wm"


Username: wiener

Password: peter

```



---



## Vulnerability



The application contains an **access control vulnerability** on the user account page.



The `userId` parameter is controlled by the user, allowing access to another user's account.




Additionally, the application leaks sensitive information in the **body of a redirect response**.



---



## Steps to Solve



### 1. Log In



Log in using:



```text id="7c8n4x"

wiener:peter

```



---



### 2. Inspect the Account Request



Open:



```text id="p3w7za"



Burp Suite → Proxy → HTTP history


```



Find the request for the account page.



Example:



```http id="r6k2vq"

GET /my-account?id=wiener

```



The important parameter is:



```text id="m8x4cp"

id=wiener

```



---



### 3. Change the User ID



Modify the parameter to request Carlos's account:



```http id="f5n9wb"

GET /my-account?id=carlos

```



The application responds with a redirect.



---




### 4. Inspect the Redirect Response



Instead of relying only on what is displayed in the browser, inspect the original response in Burp Suite.



The response body contains sensitive information even though the application performs a redirect.



The response revealed:



```text id="y8q3vd"

Your username is: carlos

```



and:



```text id="t6m1xp"

Your API Key is: asuKsO7QrWR28xm8U0lCwUenWNmyxfsG

```



---



### 5. Submit the API Key




Copy Carlos's API key and submit it as the lab solution.



The lab is solved. ✅



---



## Why Does the Vulnerability Exist?



The application trusts a **user-controlled user ID** without properly checking whether the authenticated user is 
authorized to access the requested account.




It also includes sensitive account information in the body of a redirect response.



```text id="k2v7qa"

User-controlled ID

       ↓

Request Carlos's account

       ↓


Redirect response

       ↓

Sensitive data in response body

       ↓

Carlos's API Key

```



---



## Redirect Response



A redirect response commonly uses a status code such as:



```http id="w8m3rz"

HTTP/1.1 302 Found
Location: /some-page

```



The browser usually follows the `Location` header automatically.



However, the original response may still contain a **response body**.




Therefore:



> A redirect does not necessarily mean that the response contains no sensitive information.



---



## Horizontal Privilege Escalation


This is related to **Horizontal Privilege Escalation**:



```text id="c6x9np"

Wiener

  ↓

Carlos

```



Both users have the same privilege level, but Wiener can access Carlos's information.





---



## IDOR



The vulnerability is also related to **IDOR (Insecure Direct Object Reference)** because the application uses a 
user-controlled identifier to access another user's account without properly checking authorization.



---



## Key Takeaways



* Never trust user-controlled identifiers for authorization.

* Always inspect the complete HTTP response in Burp Suite.

* Do not ignore response bodies when a redirect occurs.

* Redirect responses can contain sensitive information.



* Sensitive data should not be included in unauthorized responses.

* Authorization must be enforced server-side.

* Accessing another user's data at the same privilege level is **Horizontal Privilege Escalation**.



### Vulnerability Type



**User ID Controlled by Request Parameter with Data Leakage in Redirect**



### Related Concepts



* Access Control

* Broken Access Control

* Horizontal Privilege Escalation

* IDOR

* Information Disclosure

* HTTP Redirect

* Response Body

* Server-Side Authorization
