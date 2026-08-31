
# Lab: Referer-Based Access Control



## Objective




Log in as the regular user `wiener` and exploit the flawed access control to promote yourself to an administrator.


### Credentials



```text id="p2n8vx"

Administrator:

Username: administrator

Password: admin



Regular User:

Username: wiener

Password: peter

```




---



## Vulnerability



The application uses the **`Referer` HTTP header** as part of its access control mechanism.



The server expects requests for administrative actions to contain a `Referer` pointing to the admin panel.



For example:


```http id="m4x7cz"

Referer: https://YOUR-LAB-ID.web-security-academy.net/admin

```



The problem is that the `Referer` header is **controlled by the client**, so it cannot be trusted as proof that the 
user has administrator privileges.



---



## Steps to Solve



### 1. Log In as Administrator



Log in using:



```text id="x8v3qa"


Username: administrator

Password: admin

```



The purpose is to understand how the administrative role-change functionality works.



---



### 2. Access the Admin Panel




Open:



```text id="n6c1wp"

/admin

```



Find the functionality used to promote a user to administrator.


---



### 3. Capture the Upgrade Request


Perform the upgrade action and inspect the request in:



```text id="k5m9rd"

Burp Suite → Proxy → HTTP history

```



Look for the `Referer` header.



For example:



```http id="j2q7xs"


Referer: https://YOUR-LAB-ID.web-security-academy.net/admin

```



Send the request to **Repeater**.

---



### 4. Log In as Wiener



Log out from the administrator account and log in using:



```text id="r4v8mn"

Username: wiener

Password: peter


```



---



### 5. Replace the Session Cookie



Replace the administrator's session cookie with Wiener's session cookie.



Keep the `Referer` header pointing to the admin panel:



```http id="z6p3yw"

Referer: https://YOUR-LAB-ID.web-security-academy.net/admin

```



The request is now associated with Wiener's session while still containing the expected admin `Referer`.



---



### 6. Send the Request




Send the modified request from Burp Repeater.



Because the application incorrectly trusts the `Referer` header for authorization, the server accepts the 
administrative action.



The result is:



```text id="q9m2kc"

wiener → administrator

```



The lab is solved. ✅



---



## Why Does the Vulnerability Exist?



The application incorrectly uses the `Referer` header as part of its authorization decision.



A vulnerable check may effectively behave like:



```text id="h3w7na"

Referer = /admin?
        ↓

       Yes

        ↓

Allow administrative action

```



The correct approach is to verify the user's privileges on the server:



```text id="s8k4xp"
Request

   ↓

Identify authenticated user

   ↓

Check user's role


   ↓

Is user an Administrator?

   ↓

Yes → Allow

No  → Deny



```



---



## Why Is `Referer` Not a Security Control?




The `Referer` header indicates where the request originated from, but it does **not prove the user's identity or 
privileges**.



For example:



```http id="f5r8mq"

Referer: /admin

```





does not mean:



```text id="b7x2vc"


User = Administrator

```



The client can manipulate the header, so it should never be used as the primary authorization mechanism.



---


## Authentication vs Authorization



### Authentication



Determines **who the user is**.




```text id="n3v7qw"

Session Cookie

      ↓

Wiener

```



### Authorization



Determines **what the user is allowed to do**.




```text id="p6x1mz"

Wiener → Regular User

Administrator → Admin actions



```



In this lab, the issue is with **Authorization**.



The server knows that the request belongs to Wiener but incorrectly allows him to perform an administrator-only 

action.



---



## Attack Flow


```text id="u8k2rx"

Login as Administrator

        ↓

Perform Upgrade Action

        ↓

Capture the Request

        ↓

Notice Referer: /admin

        ↓

Send Request to Repeater

        ↓

Login as Wiener

        ↓

Replace Administrator Session


with Wiener's Session

        ↓


Keep Referer: /admin

        ↓

Send Request

        ↓

Wiener becomes Administrator

        ↓

Lab Solved ✅


```



---



## Key Takeaways



* Never use the `Referer` header as proof of authorization.

* Client-controlled headers cannot be trusted for access control.

* Authorization must be enforced server-side.

* Every privileged action should verify the current user's role.

* Authentication determines **who you are**.

* Authorization determines **what you can do**.

* When testing access control, compare requests from users with different privilege levels.

* Replaying an administrative request with a low-privileged session is a useful access control test.



### Vulnerability Type



**Referer-Based Access Control Bypass**




### Impact



**Vertical Privilege Escalation**



### Related Concepts



* Broken Access Control

* Access Control Bypass

* Vertical Privilege Escalation

* HTTP Headers

* `Referer`

* Authentication vs Authorization

* Session Management

* Server-Side Authorization

