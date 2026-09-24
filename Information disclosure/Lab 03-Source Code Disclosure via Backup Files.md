# Source Code Disclosure via Backup Files

**Difficulty:** Apprentice

**Category:** Information Disclosure / Source Code Disclosure

**Status:** Solved ✅



## Overview



**Source Code Disclosure** occurs when an application unintentionally exposes its source code to users.



A common cause is publicly accessible **backup files**, such as:



```text

.bak

.old

.backup

~

```



Backup files may contain sensitive information such as:



* Database credentials

* API keys



* Secret keys

* Internal paths

* Application logic

* Hard-coded passwords



In this lab, a publicly accessible backup file exposed Java source code containing a hard-coded PostgreSQL database password.



---



## Lab Goal



The objective was to:


> Identify and submit the database password.



---



## Step 1 — Find the Hidden Directory



We requested:



```http

GET /robots.txt HTTP/2

```



The response revealed:


```text

/backup

```



This gave us a potentially interesting hidden directory to investigate.



`robots.txt` is not an access-control mechanism. It may reveal paths that the application does not want search engines to index, but those 
paths can still be accessed directly.



---



## Step 2 — Find the Backup File



Inside the `/backup` directory, we discovered:



```text

/backup/ProductTemplate.java.bak

```



The `.bak` extension indicated that this was likely a backup copy of a Java source-code file.



---



## Step 3 — Request the Backup File



We sent:



```http

GET /backup/ProductTemplate.java.bak HTTP/2

Host: YOUR-LAB-ID.web-security-academy.net

```



The server responded with:



```http

HTTP/2 200 OK

Content-Type: text/plain; charset=utf-8

```



Because the backup file was returned as plain text, we could read the Java source code directly.



---



## Step 4 — Analyze the Leaked Source Code



The source code contained:



```java

ConnectionBuilder connectionBuilder = ConnectionBuilder.from(

    "org.postgresql.Driver",

    "postgresql",

    "localhost",

    5432,

    "postgres",
    "postgres",

    "tzslmet4x15qz05r7ys66i4vt8yx92mj"

).withAutoCommit();

```


This exposed the database connection details:



```text

Database: PostgreSQL

Host: localhost

Port: 5432

Username: postgres

Password: tzslmet4x15qz05r7ys66i4vt8yx92mj
```



The database password was:




```text

tzslmet4x15qz05r7ys66i4vt8yx92mj

```




We submitted the password and the lab was solved successfully.



---




## Why Did the Vulnerability Occur?




The root problem was not simply the existence of a backup file.



The backup file was:



1. Stored in a web-accessible directory.

2. Directly accessible without authorization.

3. Containing the application's source code.

4. Containing hard-coded database credentials.




The resulting vulnerability chain was:



```text

Exposed Backup File

        ↓


Source Code Disclosure

        ↓

Hard-coded Credentials

        ↓

Database Password Disclosure

```



---



## Key Takeaways




When testing for information disclosure, look for potentially exposed files and directories such as:



```text

/backup


.bak

.old

.backup
~

```



After obtaining source code, search for sensitive keywords such as:



```text

password

secret

key

token

database

username

API


```


Source-code disclosure can expose credentials and other internal information even when the main application does not directly reveal them.



---




## Attack Flow



```text
/robots.txt

     ↓



/backup

     ↓

ProductTemplate.java.bak

     ↓

Leaked Java Source Code

     ↓

Hard-coded Database Password

     ↓

Submit Password

     ↓

Lab Solved ✅

```
