# Username Enumeration via Response Timing



## Lab Overview



This lab is vulnerable to **username enumeration through response timing** and **password brute-force**.



The main idea is to identify a valid username by comparing response times, then brute-force the password.



---



## 1. Username Enumeration



I sent different usernames to the `/login` endpoint and compared the response times.




For invalid usernames, the response time was roughly the same.



However, when a valid username was used, the response took noticeably longer. The response time also increased 
depending on the length of the password.



```text

Invalid username → Similar response time



Valid username → Longer response time

```



This timing difference can be used to enumerate a valid username.



---



## 2. Bypassing IP-Based Protection



The lab blocks the IP after too many incorrect login attempts.



The application supports the `X-Forwarded-For` header, which can be used to spoof the client IP.



Example:



```http

X-Forwarded-For: 1

```



Then change it for each request:



```http

X-Forwarded-For: 2

X-Forwarded-For: 3

X-Forwarded-For: 4

```



This helps avoid the IP-based brute-force protection during the attack.



---



## 3. Password Brute-force



After finding the valid username, I sent the request to **Burp Intruder**.



I used a **Pitchfork attack** with two payload positions:



```http

X-Forwarded-For: §1§



username=VALID_USERNAME&password=§PASSWORD§

```

## Burp Intruder Setup


* Send the `POST /login` request to **Intruder**.

* Select **Pitchfork** as the attack type.

* Add `X-Forwarded-For` to the request.

* Mark two payload positions:



```http

X-Forwarded-For: §1§



username=VALID_USERNAME&password=§PASSWORD§

```



* **Payload 1:** `Numbers` → `1-100`, step `1`.

* **Payload 2:** Candidate passwords wordlist.

* Start the attack and check the **Status** column.

* A `302` response indicates the correct password.




### Payload 1 — X-Forwarded-For



Used the **Numbers** payload type:



```text

From: 1

To: 100

Step: 1

```



### Payload 2 — Password



Used the **Candidate passwords** list.



---



## 4. Finding the Correct Password



After the attack finished, I compared the responses.



The correct password returned a:



```text

302

```



response, which indicated a successful login attempt.



---



## 5. Final Login



I used the discovered username and password to log in through the normal login page.



The first login attempt was blocked because the IP had already been temporarily locked due to previous incorrect 
attempts.



After the lockout expired, I could log in using the correct credentials and access the account page.



---



## Attack Flow


```text

Response Timing

      ↓

Username Enumeration

      ↓

Valid Username

      ↓

X-Forwarded-For

      ↓

Password Brute-force

      ↓

302 Response

      ↓

Valid Password

      ↓

Account Access

```



## Key Takeaways



* Username enumeration can be based on **response timing**, not only different error messages.

* A valid username may cause a noticeably longer response time.

* `X-Forwarded-For` can be abused to bypass IP-based brute-force protection when it is trusted by the application.

* During password brute-force, a successful login can be identified by a different status code such as `302`.

* Enumerating the username first is generally more efficient than brute-forcing usernames and passwords together.

