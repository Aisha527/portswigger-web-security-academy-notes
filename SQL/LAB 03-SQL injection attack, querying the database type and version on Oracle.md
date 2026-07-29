# SQL injection attack, querying the database type and version on Oracle

## Concept


The application contains a SQL Injection vulnerability in the product category filter.



The goal is to use a UNION attack to retrieve the database version.


---


## Vulnerability



User input is directly inserted into the SQL query:



```sql

SELECT * FROM products 

WHERE category = 'Gifts' AND released = 1

````


The category parameter can be manipulated to inject another SQL query.


---


## Exploitation


### 1. Confirm SQL Injection


Test with:


```sql

'

```


The error confirms that the input affects the SQL query.


---


### 2. Determine number of columns


Use UNION with NULL values:


```sql

' UNION SELECT NULL--

```


Increase the number of NULL values until the query works.


Example:


```sql

' UNION SELECT NULL,NULL--

```

---


### 3. Retrieve database version


The database type is Oracle.


Oracle requires every SELECT statement to have a FROM clause.


Use the built-in table:


```sql

dual

```


Database version is stored in:


```sql

v$version

```


The version string is in:


```sql

banner

```


Payload example:



```sql

' UNION SELECT banner FROM v$version--

```


(Adjusted depending on the number of columns.)


---


## Key Takeaways


* UNION attacks combine the original query result with an injected query.

* The number of columns must match the original query.

* Each DBMS has different methods to retrieve information.

* Oracle requires FROM in SELECT statements.

* `dual` is a built-in Oracle table used for simple queries.


---


## Database Version Queries


### Oracle:


```sql

SELECT banner FROM v$version

```


### MySQL:


```sql

SELECT @@version

```


### PostgreSQL:



```sql

SELECT version()

```


### SQL Server:


```sql

SELECT @@version

```

```
