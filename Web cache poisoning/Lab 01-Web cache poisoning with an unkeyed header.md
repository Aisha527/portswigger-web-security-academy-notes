# Web Cache Poisoning with an Unkeyed Header

## Lab Notes — PortSwigger Web Security Academy



> **Lab:** Web cache poisoning with an unkeyed header

> **Source:** PortSwigger Web Security Academy — official lab solution

> **Vulnerability class:** Web Cache Poisoning via unkeyed `X-Forwarded-Host` header



---



## 🔎 What is Web Cache Poisoning?



**Web Cache Poisoning** is a vulnerability where an attacker manipulates a

caching layer (a CDN, reverse proxy, or built-in web cache) into storing a



malicious or attacker-controlled HTTP response, so that this poisoned response

is then served to **other, legitimate users** who request the same cached

resource.



It typically happens when:




- The backend application includes some part of the request (a header, cookie,


  or parameter) in the response it generates — often without properly

  validating or sanitizing it.

- The cache decides whether two requests are "the same" using a **cache key**,

  which usually only includes the URL/path — **not every header or parameter**.

- If an input that affects the response is *not* part of the cache key (an

  "unkeyed input"), an attacker can send one malicious request, get a




  malicious response cached, and that same cached response will then be

  delivered to every other visitor — no direct access to their browser needed.



Impact ranges from defacement to stored XSS, open redirects, or serving

malicious JavaScript to every visitor of a page, as demonstrated below.



---



## Concept of This Lab



A caching layer sits in front of the origin server. The cache decides whether

two requests are "the same" based on a **cache key** — usually just the

URL/path, *not* every header. If the backend uses a header (like

`X-Forwarded-Host`) to build part of the response, but the cache does **not**


include that header in its key, an attacker can poison the cached response for

*every other visitor*.


## Steps



| # | Action | Purpose |

|---|--------|---------|

| 1–2 | Load the home page, send the request to Repeater | Get a baseline request to work with |

| 3 | Add a cache buster: `GET /?cb=1234` | Force a fresh (non-cached) response for testing, avoid polluting real cache 
while probing |

| 4–5 | Add `X-Forwarded-Host: example.com` | Test whether the backend reflects this header into the response (it did — 
it appeared inside a `<script src="//example.com/resources/js/tracking.js">` tag) |

| 6 | Replay the same request | Check response headers for `X-Cache: hit` → confirms the poisoned response was served 
**from cache**, not freshly generated |

| 7 | On the Exploit Server, create a file named `/resources/js/tracking.js` with body `alert(document.cookie)` | Host a 
malicious JS payload at the exact path the vulnerable page requests |

| 8 | Store the exploit, note the exploit server URL | Needed for the next step |

| 9 | Remove the cache buster → `GET /` | Target the **real** cache entry that real visitors will hit |

| 10 | Set `X-Forwarded-Host` to the exploit server's domain and send | Poison the real cache entry so the injected 
`<script>` now points to the attacker's file |



## Why It Works



```

X-Forwarded-Host (attacker-controlled)

        ↓

Reflected into HTML response (unsafe backend logic)

        ↓

Cache stores the response, but does NOT key on X-Forwarded-Host

        ↓

Every subsequent visitor to "/" gets the poisoned response

        ↓

Malicious tracking.js loads from the exploit server → XSS for all visitors

```



## Key Takeaway



This is a classic **unkeyed input** cache poisoning bug. Any header, cookie, or

parameter that influences the response body but is excluded from the cache key

is a potential poisoning vector. Defenses: don't let untrusted headers

influence rendered output, or ensure the cache key includes every input that

affects the response (or disable caching for such responses).


---



## 📌 Quick Reference



- **Vulnerable header:** `X-Forwarded-Host`

- **Reflected into:** `<script src="//<host>/resources/js/tracking.js">`

- **Cache confirmation header:** `X-Cache: hit`

- **Payload file path:** `/resources/js/tracking.js`

- **Payload body:** `alert(document.cookie)`

- **Official reference:** PortSwigger Web Security Academy — *Web cache poisoning with an unkeyed header*
