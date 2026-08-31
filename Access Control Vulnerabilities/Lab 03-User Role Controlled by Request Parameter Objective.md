# Lab: User Role Controlled by Request Parameter

## Objective

Access the admin panel and use it to delete the user `carlos`.



### Credentials



```text

Username: wiener

Password: peter

```



---


## Vulnerability



The application uses a **client-controlled cookie** to determine whether a user has administrator privileges.



The cookie can be modified by the user, and the server trusts its value without properly verifying the user's actual 
role.



---




## Steps to Solve



### 1. Log in



Log in using:



```text

wiener:peter

```



---



### 2. Inspect the Cookies



Open the browser's Developer Tools:



```text

Inspect → Application → Cookies

```



Or inspect the request using Burp Suite.



Find the administrator-related cookie:



```http
Cookie: Admin=false

```



---



### 3. Modify the Cookie



Change:



```text

Admin=false

```


to:



```text

Admin=true

```



The modified cookie makes the application treat the user as an administrator.

---


### 4. Access the Admin Panel



Navigate to:



```text

/admin

```

The admin panel should now be accessible.



---



### 5. Delete Carlos


From the admin panel:




```text

carlos → Delete

```



The lab is solved.


---

## Why Does the Vulnerability Exist?

The application makes an authorization decision based on data controlled by the client:



```text

Client

  ↓


Admin=false

  ↓

Server trusts the value

```



An attacker can modify the value:



```text

Admin=false

      ↓

Admin=true

```




The server then incorrectly grants administrator privileges.



---





## Attack Flow



```text

Normal User


     ↓

Admin=false

     ↓

Modify Cookie

     ↓

Admin=true

     ↓

/admin


     ↓

Admin functionality

```



---



## Privilege Escalation




This is an example of **Vertical Privilege Escalation**:



```text

Normal User → Administrator

```



The attacker moves from a lower privilege level to a higher privilege level.



---



## Key Takeaways



* Never trust client-controlled data for authorization decisions.

* Cookies can be modified by the user.

* Client-side values should not determine security privileges.

* Authorization must be properly enforced on the server side.

* Hidden or client-controlled role information is not a security mechanism.



### Vulnerability Type



**User Role Controlled by Request Parameter**



### Related Concepts



* Access Control

* Broken Access Control

* Client-Controlled Data

* Cookie Manipulation

* Vertical Privilege Escalation

* Server-Side Authorization


