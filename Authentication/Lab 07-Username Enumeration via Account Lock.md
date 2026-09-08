# Username Enumeration via Account Lock



## 🎯 Lab Objective



This lab demonstrates a **username enumeration** vulnerability caused by flawed account-locking logic.



The goal is to:



1. Identify a valid username.

2. Brute-force the password for that username.

3. Log in to the account and solve the lab.



---



## 🔍 Vulnerability Overview



The application uses account locking after several failed login attempts.


Normally, this should make brute-force attacks more difficult. However, the application has a logic flaw that allows us 
to distinguish a valid username from invalid usernames.



For invalid usernames, the application returns an error such as:



```text

Invalid username or password.

```



For a valid username that has triggered the account lock, the response contains:



```text

You have made too many incorrect login attempts.

```



This difference can be used for **username enumeration**.



---



# 1. Username Enumeration



I sent the login request to **Burp Intruder**.



The request was configured as:



```http

POST /login

```



### Attack Type



```text

Cluster bomb

```



### Payload Positions



The username was marked as the first payload position, and an additional blank position was added to the end of the 
request:



```text

username=§invalid-username§&password=example§§

```



This created two payload positions:



```text

Payload 1 → Username

Payload 2 → Number of attempts

```



---



## Payload Set 1 — Usernames



I used the candidate username list as:



```text

Simple list

```



---



## Payload Set 2 — Repeated Attempts



For the second payload position, I used:



```text

Null payloads

```



and generated:




```text

5 payloads

```


This causes every username to be tested five times consecutively.



For example:



```text

user1

user1

user1

user1

user1


user2

user2

user2

user2

user2


```



With 101 usernames:



```text

101 × 5 = 505 requests
```



---



# 2. Alternative Python Method




The usernames can also be prepared using Python:



```python

with open("usernames.txt", "r") as file:

    usernames = [line.strip() for line in file if line.strip()]



for username in usernames:

    for _ in range(5):

        print(username)

```




This produces a list where each username appears five times consecutively.



The resulting list can then be loaded into Burp Intruder as a payload list.



---



# 3. Identifying the Valid Username



After starting the attack, I compared the responses.



Most responses were similar, but one username produced a noticeably longer response.



The response contained:



```text

You have made too many incorrect login attempts.

```



The valid username was:



```text

amarillo


```



---



# 4. Password Brute Force



I created a new Intruder attack using the same login request.



### Attack Type





```text



Sniper

```



The username was fixed to:



```text

amarillo

```



I then marked the password as the payload position:



```text

username=amarillo&password=§example§
```



I loaded the candidate password list and created a **Grep - Extract** rule for the login error message.



---


# 5. Identifying the Password




Most password attempts returned the expected error message.




However, one response did not contain the error message.


The password was:




```text

computer


```



Therefore, the valid credentials were:



```text

Username: amarillo

Password: computer

```



---



# 6. Login and Lab Completion



After waiting for the account lock to reset, I logged in using the discovered credentials:


```text

Username: amarillo

Password: computer

```



I successfully accessed the account page and solved the lab.



---



## 🧠 Key Takeaways






* Different error messages can enable **username enumeration**.

* **Response length** can be a useful indicator when analyzing Intruder results.


* **Cluster bomb** can be used to combine multiple payload sets.

* **Null payloads** can generate repeated requests without modifying the parameter value.

* **Sniper** is useful when testing a single payload position against a fixed username.

* Account locking does not necessarily prevent username enumeration if the locking mechanism contains a **logic flaw**.

* When analyzing Intruder results, compare:




  * Response length

  * Error messages

  * Response content

  * Status codes



## Vulnerability



```text

Username Enumeration

+

Flawed Account Locking Logic

```



## Tools


* Burp Suite

* Burp Intruder

* Python

* PortSwigger Web Security Academy

