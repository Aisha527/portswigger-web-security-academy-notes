# Username Enumeration via Subtly Different Responses

## Lab Idea



This lab is vulnerable to **username enumeration** and **password brute-force**.



The login page gives a slightly different response when a valid username is submitted. Even though the error message 
looks almost identical, the difference can be detected by comparing the responses.



### Goal



1. Enumerate a valid username.

2. Brute-force the password for that username.

3. Log in and access the account page.




---



## 1. Username Enumeration



I sent the **Candidate usernames** wordlist to the login endpoint and compared the responses.



Most usernames returned the same error message.



For the valid username:



```text

adm

```



the error message looked the same, but there was a small difference at the end:



```text

Invalid username or password.</p>


```



while the normal response did not contain this extra:



```html

</p>

```



So I used this subtle difference to identify:



```text

Username: adm

```



### Important



The vulnerability wasn't a different error message.



The application basically returned the **same error**, but the response had a small difference in the HTML:



```html

</p>

```



This was enough to distinguish a valid username from invalid ones.



---



## 2. Password Brute-force



After finding the valid username, I used the **Candidate passwords** wordlist while keeping the username fixed:



```text

Username: adm


```



The correct password was:



```text

michael

```



So the final credentials were:



```text

Username: adm

Password: michael

```




---



## 3. Login



I logged in using:



```text

adm:michael

```



and accessed the account page, which completed the lab.



---



## Vulnerability



This is a **username enumeration** vulnerability caused by subtly different server responses.



Even a very small difference in the response can be used to identify valid usernames.



The general attack flow is:



```text

Username Enumeration

        ↓

Find valid username

        ↓

Password Brute-force

        ↓

Valid credentials

        ↓

Account access

```



## Key Takeaways



* Error messages don't have to be obviously different for username enumeration to work.

* Always compare the full HTTP responses, not just the visible error message.

* Small differences such as HTML characters, response length, status codes, or response timing can reveal a valid 
username.

* Once a valid username is found, password brute-forcing becomes much easier.

