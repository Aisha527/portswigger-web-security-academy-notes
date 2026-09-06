# Authentication Vulnerabilities


## Definition



**Authentication vulnerabilities** are weaknesses in the mechanisms used to verify a user's identity.



They may allow an attacker to:



* Identify valid usernames.

* Brute-force passwords.

* Bypass login mechanisms.

* Gain unauthorized access to user accounts.



### Common Examples



* Username Enumeration

* Brute-Force Attacks

* Password Reset Vulnerabilities

* MFA Vulnerabilities



---



# Vulnerabilities in Password-Based Login



These vulnerabilities affect authentication mechanisms that rely on:



```text

Username + Password

```



Common examples include **Username Enumeration** and **Password Brute-Force**.



---



# Username Enumeration



## Definition



**Username Enumeration** is a vulnerability that allows an attacker to determine whether a particular username exists 
in an application.




This can happen when the application produces different responses for valid and invalid usernames.



The difference may appear in:



* Error messages

* Response body

* Response length

* Status code

* Application behavior


---



# Lab: Username Enumeration via Different Responses



## Goal



1. Enumerate a valid username.

2. Brute-force the user's password.

3. Log in and access the account page.



---



## 1. Enumerate a Valid Username



Send a login request:



```http

POST /login

```



with:



```text

username=USERNAME

password=PASSWORD

```



Send the request to **Burp Intruder**.



Set the payload position on the username:



```text

username=§USERNAME§&password=PASSWORD

```



Load the:



```text

Candidate usernames

```



wordlist.


Compare the responses.



Most invalid usernames return the same response, while a valid username produces a **different response**.



For example:



```text


Invalid username or password

```



vs.



```text

Incorrect password

```



The username that produces the different response is the **valid username**.



---



## 2. Brute-Force the Password


Once the valid username is identified, keep it fixed:



```text

username=VALID_USERNAME

```



Set the payload position on the password:



```text

username=VALID_USERNAME&password=§PASSWORD§

```


Load the:


```text

Candidate passwords

```



wordlist.



Compare the responses to identify the **valid password**.



---



## 3. Log In


Use:



```text

Valid Username + Valid Password

```



Then log in and access the **Account page**.



---



# Why Did the Attack Work?



The lab contains two weaknesses:



### 1. Username Enumeration



The application returns **different responses** for valid and invalid usernames.



This allows an attacker to identify valid usernames.



### 2. Weak Brute-Force Protection



The application does not sufficiently restrict repeated login attempts, allowing the attacker to try many candidate 
passwords.



---


# Key Notes ⭐



* **Authentication** → Verifies *who you are*.

* **Username Enumeration** → Identifies whether a username exists.

* **Different Responses → Username Enumeration**

* Do not rely only on **status code or response length**. Check the response content as well.

* After finding a valid username, attempt password brute-forcing.

* **Username Enumeration ≠ Brute Force**.

* **Rate limiting** helps prevent brute-force attacks.

* Successful exploitation may lead to **Account Takeover**.



## Attack Flow



```text

Candidate Usernames
        ↓

Compare Responses

        ↓

Valid Username

        ↓

Candidate Passwords

        ↓

Valid Password

        ↓

Login

        ↓
Account Access

```



## Vulnerability



**Username Enumeration via Different Responses**



### Impact



Reveals valid usernames and makes password brute-force attacks easier.

