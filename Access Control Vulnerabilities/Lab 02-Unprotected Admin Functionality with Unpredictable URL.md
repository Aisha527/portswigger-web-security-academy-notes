# Lab: Unprotected Admin Functionality with Unpredictable URL


## Objective


Access the hidden admin panel and delete the user `carlos`.


---



## Vulnerability



The application has an **unprotected admin panel** located at an unpredictable URL.


Although the URL is difficult to guess, the application discloses it in the **client-side JavaScript**.



The server also fails to properly enforce authorization, allowing a normal user to access the admin panel directly.



---



## Steps to Solve



### 1. Inspect the Page Source



Open:




```text

View Page Source

```



Search for:



```text

admin

```



We find:



```javascript

var isAdmin = false;


if (isAdmin) {

   var topLinksTag = document.getElementsByClassName("top-links")[0];

   var adminPanelTag = document.createElement('a');
   adminPanelTag.setAttribute('href', '/admin-8fgjsw');

}

```



### 2. Discover the Admin Panel URL



The important line is:



```javascript

adminPanelTag.setAttribute('href', '/admin-8fgjsw');

```



This reveals the admin panel path:



```text

/admin-8fgjsw

```


### 3. Access the Admin Panel



Navigate directly to:



```text

/admin-8fgjsw

```


Even though:


```javascript

isAdmin = false

```


the application still allows access to the admin panel.



### 4. Delete Carlos



From the admin panel:



```text

carlos → Delete

```



The lab is solved.



---



## Why Does the Vulnerability Exist?



The application uses **client-side JavaScript** to hide the admin panel link:



```javascript

if (isAdmin) {

    // Show Admin Panel link

}

```



However, hiding a link does not prevent a user from accessing the endpoint directly.



The server should perform an authorization check for every sensitive request.



---



## Client-Side vs Server-Side Authorization



### ❌ Insecure



```text

User → Hidden Admin URL → Admin Panel

```



### ✅ Secure



```text

User → Admin URL

          ↓

   Server checks role

          ↓

   Admin? → Allow

   User?  → Deny

```



Client-side code should **never be relied upon as the only access control mechanism**.



---



## Key Takeaways



* Hidden URLs are not a security control.

* Unpredictable URLs do not replace proper authorization.

* Client-side JavaScript can reveal hidden endpoints.

* Authorization must be enforced **server-side**.

* A normal user accessing admin functionality is an example of **Vertical Privilege Escalation**.



### Vulnerability Type



**Unprotected Admin Functionality**



### Related Concept



**Vertical Privilege Escalation**
