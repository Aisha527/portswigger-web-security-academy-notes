# Lab: Brute-forcing a stay-logged-in cookie

## 🔹 Lab Objective



The `stay-logged-in` cookie is used to keep a user authenticated after closing the browser.

In this lab, the cookie is generated using predictable information, making it vulnerable to brute-forcing.

The goal is to brute-force Carlos's `stay-logged-in` cookie and access his **My account** page.



---



## 🔹 1. Log in to our account



Credentials:



```text

Username: wiener

Password: peter

```



With Burp running, log in to the lab with **Stay logged in** enabled.



This creates a `stay-logged-in` cookie.



---



## 🔹 2. Analyze the Cookie



Inspect the `stay-logged-in` cookie in Burp's **Inspector**.



After Base64 decoding the cookie, we get:



```text

wiener:51dc30ddc473d43a6011e9ebba6ca770

```



The second part looks like an MD5 hash because of its length and hexadecimal character set.



Hashing our password:



```text

MD5(peter)

```



produces the same hash.



Therefore, we can determine that the cookie is constructed as:



```text

Base64(username + ":" + MD5(password))

```



For example:



```text


wiener:MD5(password)

        ↓

     Base64

        ↓

stay-logged-in cookie

```



---



## 🔹 3. Send the Request to Intruder



Log out of the account.



Find the latest request:



```http

GET /my-account?id=wiener

```



Highlight the `stay-logged-in` cookie value and send the request to **Burp Intruder**.



---



## 🔹 4. Configure the Payload



Burp automatically identifies the `stay-logged-in` value as a payload position.



First, use our own password as a single payload:



```text

peter

```



This is used to verify that our payload-processing rules work correctly.



---



## 🔹 5. Payload Processing



Under:



```text

Payloads → Payload processing

```



add these rules in this exact order:



### 1. Hash



```text

MD5

```



### 2. Add Prefix


```text

wiener:

```



### 3. Encode



```text

Base64-encode


```



The payload is processed sequentially:




```text

peter

 ↓

MD5(peter)

 ↓

wiener:MD5(peter)

 ↓

Base64

 ↓

stay-logged-in cookie

```



---



## 🔹 6. Verify the Processing



The **Update email** button is only displayed when the user is authenticated on the My account page.




Therefore, it can be used to identify a successful response.



Go to:



```text
Settings → Grep - Match

```



Add:




```text

Update email

```



Then start the attack.



The generated payload should successfully load our own account page.



This confirms that the payload-processing rules are working correctly.



---



## 🔹 7. Attack Carlos's Account



Now modify the attack.



### Payload list


Remove:



```text

peter

```



and replace it with the list of **candidate passwords**.


### URL



Change:



```text

/my-account?id=wiener

```



to:


```text
/my-account?id=carlos

```



### Prefix



Change:



```text

wiener:

```



to:



```text

carlos:

```



The processing is now:



```text

candidate password

        ↓

      MD5

        ↓

carlos:MD5(password)

        ↓

     Base64
        ↓

stay-logged-in cookie

```



---




## 🔹 8. Analyze the Results



Start the attack.



In this attack, the responses can all return:


```text

302

```



So the status code alone is not enough to identify the valid cookie.



Compare other characteristics of the responses, such as:



* Response length

* Response body


* `Location` header

* Grep - Match results

In my attack, the responses were all `302`, but one response had a different length:



```text

1340

```



This was the successful response.




The lab was then marked as **Solved**.



---



## 💡 Key Takeaways




* A `stay-logged-in` cookie can act as an authentication mechanism.


* Predictable cookie construction can make it vulnerable to brute-force attacks.

* Base64 is an encoding, not encryption.

* Payload Processing in Burp Intruder can transform each password automatically.

* The order of Payload Processing rules matters.

* A `302` response does not necessarily mean that all attempts behaved identically.

* When status codes are the same, compare the **response length, body, headers, or specific strings** to identify the 
successful request.

