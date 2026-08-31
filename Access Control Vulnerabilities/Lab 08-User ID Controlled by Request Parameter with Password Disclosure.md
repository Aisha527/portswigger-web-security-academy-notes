# Lab: User ID Controlled by Request Parameter with Password Disclosure


## Objective



Retrieve the administrator's password, log in as the administrator, and use the admin panel to delete the user 
`carlos`.



### Credentials




```text

Username: wiener

Password: peter

```



---



## Vulnerability



The application contains an **access control vulnerability** in the user account page.




The `User ID` can be controlled through the request, allowing a normal user to access another user's account page.



The account page also contains the user's current password inside a masked password input.



This allows us to retrieve the administrator's password.



---



## Steps to Solve



### 1. Log In



Log in with:



```text

wiener:peter

```



---



### 2. Identify the User ID Parameter



Open **My Account** and inspect the request using:



```text

Burp Suite → Proxy → HTTP history

```



Find the request that contains the user identifier.





For example:



```http

GET /my-account?id=wiener

```



---



### 3. Access the Administrator Account



Modify the user ID from:



```text

id=wiener

```



to:




```text

id=administrator

```



Send the modified request.




The application returns the administrator's account page without properly checking authorization.



---



### 4. Find the Administrator's Password



Inspect the HTML source or the response.



The password is stored inside a password input:



```html

<input type="password" name="password" value="d46983lzkfvu6u6n4vso">

```



The password is:


```text

d46983lzkfvu6u6n4vso

```



---



### 5. Log In as Administrator



Use the discovered credentials:


```text

Username: administrator

Password: d46983lzkfvu6u6n4vso

```



---



### 6. Delete Carlos



Open:



```text

/admin

```



Then delete:



```text

carlos

```


The lab is solved. ✅

---



## Why Does the Vulnerability Exist?



The application trusts a **user-controlled User ID** and does not properly verify whether the current user is 
authorized to access the requested account.



It also exposes the user's current password in the HTML.



```text

User-controlled ID

       ↓
Administrator account

       ↓

Password disclosed in HTML

       ↓

Login as Administrator

       ↓

Delete Carlos

```



---


## Password Masking



The following:



```html

<input type="password" value="...">

```



does **not** protect the password value.



`type="password"` only controls how the value is displayed in the browser.



The actual value is still present in the HTML:



```html

value="password"

```



Therefore, a user who can access the page can potentially read the password from the source or HTTP response.



---



## Privilege Escalation



This results in **Vertical Privilege Escalation**:



```text

Normal User

     ↓

Administrator

```



The attacker moves from a lower privilege level to a higher privilege level.



---



## CSRF Token



The page also contained a hidden CSRF token:



```html

<input type="hidden" name="csrf" value="...">

```



The CSRF token is not the vulnerability being exploited in this lab.



The main issues are:



* Broken access control

* User ID manipulation

* Password disclosure



---



## Attack Flow



```text

Wiener

   ↓

Modify User ID

   ↓

Access Administrator Account

   ↓
Read Password from HTML

   ↓


Login as Administrator

   ↓
Access /admin

   ↓

Delete Carlos

```



---



## Key Takeaways




* Never trust user-controlled identifiers for authorization.

* Authorization must be enforced server-side.

* Never expose plaintext passwords in HTML or HTTP responses.

* `type="password"` only masks the value visually.

* Always inspect the HTML and HTTP responses when testing access control.

* Sensitive credentials should never be returned to the client.

* Accessing an administrator account from a normal account can lead to **Vertical Privilege Escalation**.



### Vulnerability Type



**User ID Controlled by Request Parameter with Password Disclosure**



### Related Concepts

* Access Control

* Broken Access Control

* IDOR

* Parameter Tampering

* Password Disclosure

* Information Disclosure

* Vertical Privilege Escalation

* Server-Side Authorization

* CSRF
