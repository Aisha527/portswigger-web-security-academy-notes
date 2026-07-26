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