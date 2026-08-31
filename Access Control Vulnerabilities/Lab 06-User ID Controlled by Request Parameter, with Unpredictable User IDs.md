# Lab: User ID Controlled by Request Parameter, with Unpredictable User IDs


## Objective



Find the GUID of the user `carlos`, obtain his API key, and submit it as the solution.



### Credentials




```text

Username: wiener

Password: peter

```



---



## Vulnerability



The application has a **horizontal privilege escalation** vulnerability on the user account page.



Users are identified by unpredictable **GUIDs** instead of simple usernames or sequential IDs.




Although the GUID is difficult to guess, the application discloses Carlos's GUID in another part of the application.





The server also fails to verify whether the authenticated user is authorized to access the requested account.



---



## Steps to Solve



### 1. Log In



Log in using:



```text

wiener:peter

```



---







### 2. Find Carlos's GUID



Open a blog post and inspect its source code:



```text

View Page Source

```



Search for:



```text

carlos

```



We found:


```html

<a href='/blogs?userId=8785bc36-c172-421c-b803-a865fc4a1ef4'>carlos</a>

```



This reveals Carlos's GUID:



```text

8785bc36-c172-421c-b803-a865fc4a1ef4

```



---



### 3. Modify the User ID


The account page uses the `userId` parameter to identify the requested account.



Change the existing user ID to Carlos's GUID:



```http

GET /my-account?id=8785bc36-c172-421c-b803-a865fc4a1ef4

```



---



### 4. Access Carlos's Account



The application returns Carlos's account information without checking whether `wiener` is authorized to access it.



---



### 5. Obtain the API Key



Find the API key displayed on Carlos's account page and submit it as the lab solution.



The lab is solved. ✅




---



## Why Does the Vulnerability Exist?



The application relies on a user-controlled `userId` parameter to determine which account to display.



The GUID is unpredictable, but it is disclosed elsewhere in the application.



The main security issue is the missing **server-side authorization check**.



```text
Wiener

   ↓

Find Carlos's GUID

   ↓


Modify userId

   ↓

Access Carlos's account


   ↓

Obtain API Key

```



---



## Horizontal Privilege Escalation



This is an example of **Horizontal Privilege Escalation**:



```text

User A (wiener)

       ↓

User B (carlos)

```



Both users have the same privilege level, but `wiener` can access `carlos`'s data.



---



## IDOR



This vulnerability is related to **IDOR (Insecure Direct Object Reference)**.



The application uses a direct object identifier such as:


```text

userId

```



to access a resource without properly checking whether the current user is authorized to access it.



---



## Unpredictable IDs Are Not Access Control



Using unpredictable GUIDs can make IDs harder to guess, but it does not replace proper authorization.



```text

Predictable ID
    ↓

Hard to guess? ❌

```



```text

Unpredictable GUID

    ↓

Harder to guess? ✅

    ↓

Authorization check still required

```



If the GUID can be discovered through another feature, the vulnerability can still be exploited.



---



## Key Takeaways



* Unpredictable IDs do not replace authorization.

* Always test user-controlled identifiers such as `id`, `userId`, and `accountId`.

* Look for ID disclosure in other parts of the application.

* Never trust a user-controlled identifier to authorize access.

* Authorization must be enforced server-side.

* Accessing another user's data at the same privilege level is **Horizontal Privilege Escalation**.



### Vulnerability Type



**User ID Controlled by Request Parameter with Unpredictable User IDs**



### Related Concepts



* Access Control

* Broken Access Control

* Horizontal Privilege Escalation


* IDOR

* GUID

* Information Disclosure

* Parameter Tampering

* Server-Side Authorization

