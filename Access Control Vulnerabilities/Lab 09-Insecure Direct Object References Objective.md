# Lab: Insecure Direct Object References


## Objective



Find the password of the user `carlos` and use it to log in to his account.



---



## Vulnerability



The application stores user chat logs directly on the server's file system.



The transcripts are accessed using static URLs containing a file name:



```text

/download-transcript/1.txt

```



The application does not properly verify whether the current user is authorized to access the requested transcript.




This creates an **Insecure Direct Object Reference (IDOR)** vulnerability.



---



## Steps to Solve



### 1. Access the Chat



Open the chat functionality:




```text

/chat

```



Start or view a conversation.



---



### 2. Inspect the Request



Open:



```text

Burp Suite → Proxy → HTTP history

```



Look for a request used to download a chat transcript.



Example:



```http

GET /download-transcript/1.txt

```



The important part is:



```text

1.txt

```



This is a direct reference to a file stored on the server.



---



### 3. Read the Transcript



The server returns the transcript:



```http

HTTP/2 200 OK

Content-Type: text/plain

```



For example:



```text

CONNECTED: -- Now chatting with Hal Pline --

```



The transcript contained a password:



```text

xlop572vtvnbvl5ujemw

```



However, this was Hal Pline's password, not Carlos's.



---



### 4. Change the File Reference


Modify the file name in the URL:



```text

/download-transcript/1.txt

```



Try other transcript numbers, for example:



```text

/download-transcript/2.txt

/download-transcript/3.txt

```



Continue checking the responses until you find the transcript belonging to `carlos`.



---



### 5. Find Carlos's Password



Inspect the transcript containing Carlos's conversation.



Look for his password in the chat content.



---



### 6. Log In as Carlos



Use the discovered password:


```text

Username: carlos

Password: [password found]

```



The lab is solved. ✅



---



## Why Does the Vulnerability Exist?




The application uses a user-controlled file reference:



```text

/download-transcript/1.txt

```



but does not perform a proper authorization check before returning the file.



An attacker can change the file reference and access another user's transcript.



```text

User's transcript

      ↓

Change file ID

      ↓

Another user's transcript

      ↓

Sensitive information

      ↓

Carlos's password

```



---



## What Is IDOR?



**IDOR (Insecure Direct Object Reference)** occurs when an application exposes a direct reference to an internal 
object and fails to properly verify whether the current user is authorized to access it.



Common examples include:



```text

User ID

Account ID

File ID

Order ID

Document ID
```



In this lab, the vulnerable object is a **chat transcript file**.




---



## Important Point



The problem is not simply that the filename is predictable.



The real problem is the missing **server-side authorization check**.



The server should verify:



```text

Who is requesting the file?

        ↓

Which file are they requesting?

        ↓

Is this user authorized to access it?

        ↓

Yes → Return file



No  → Deny access

```



---



## Impact



An IDOR vulnerability can allow attackers to access sensitive information belonging to other users.



In this lab:



```text

Unauthorized transcript access

        ↓

Password disclosure

        ↓

Account takeover



```



---



## Key Takeaways



* Always test direct object references in URLs and requests.

* Try modifying IDs, filenames, and other object references.

* Files stored on the server also require authorization checks.

* Predictable object identifiers can make IDOR easier to discover.

* Unpredictable identifiers alone are not a security control.

* Authorization must be enforced server-side.

* IDOR can lead to sensitive information disclosure and account takeover.



### Vulnerability Type



**Insecure Direct Object References (IDOR)**




### Related Concepts


* Broken Access Control

* Horizontal Privilege Escalation

* IDOR

* Sensitive Information Disclosure

* File Access Control

* Direct Object References



* Server-Side Authorization

* Account Takeover
