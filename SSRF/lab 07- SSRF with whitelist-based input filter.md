# Lab: SSRF with whitelist-based input filter

## Description

The application uses a whitelist to restrict SSRF requests to trusted hosts.

The objective is to bypass the whitelist and access the internal admin interface.

---

## Objective

- Access `http://localhost/admin`
- Delete the user `carlos`

---

## Protection

The application validates the destination host using a **Whitelist**.

Only trusted domains are allowed.

---

## Vulnerability

The application parses URLs incorrectly.

The security filter and the HTTP client interpret the URL differently, allowing the whitelist validation to be bypassed.

---

## Exploitation

The attack abuses the URL **userinfo** section (`userinfo@host`) together with **double URL encoding**.

Example payload:

```text
http://localhost:80%2523@stock.weliketoshop.net/admin/delete?username=carlos
```

---

## Why it works

- The whitelist validates the URL before full decoding.
- The HTTP client decodes and interprets the URL differently.
- This parser inconsistency causes the request to be sent to the internal server instead of the trusted host.

---

## Key Takeaways

- Whitelists are stronger than blacklists but can still fail if URL parsing is inconsistent.
- Different components may interpret the same URL differently.
- URL parser confusion is a common SSRF bypass technique.
- Double URL Encoding can be used to exploit parser inconsistencies.

---

## References

- https://portswigger.net/web-security/ssrf
- https://portswigger.net/web-security/ssrf/lab-ssrf-with-whitelist-filter