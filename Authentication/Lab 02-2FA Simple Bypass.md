# Authentication Vulnerabilities


## 2FA Simple Bypass



### Definition



**Two-Factor Authentication (2FA)** adds an additional verification step after the username and password.



A secure authentication flow should be:



```text

Username + Password

        ↓

2FA Verification

        ↓

Authenticated Session

        ↓

Account Access

```



A **2FA bypass vulnerability** occurs when an attacker can access protected resources without successfully completing 
the second authentication factor.



---



# Lab: 2FA Simple Bypass



## Goal



Access **Carlos's account page** without having access to his 2FA verification code.



### Credentials


```text

Wiener:

Username: wiener

Password: peter



Carlos:

Username: carlos

Password: montoya

```



---



## Exploitation



### 1. Log in as Carlos



Use:



```text

Username: carlos

Password: montoya

```



The application then asks for a **2FA verification code**.



---


### 2. Skip the 2FA step


Instead of providing the 2FA code, directly request the account page:



```text

/my-account

```



The application incorrectly allows access to the account page without verifying that the 2FA step was completed.



---



## Attack Flow



```text

Valid Username + Password

          ↓

       2FA Required

          ↓

      Skip 2FA

          ↓
      /my-account

          ↓

   Account Access

```



---



# Root Cause




The vulnerability is caused by incorrect handling of the **authentication flow**.




The server does not properly verify that the user has completed the 2FA step before granting access to protected 
resources.



The application should perform a **server-side check** before allowing access to the account.



---



# Key Notes ⭐



* **2FA** = An additional authentication factor after the password.

* 2FA must be enforced **server-side**.

* Reaching the 2FA page does not mean that 2FA has been successfully completed.

* Protected resources must verify the user's **fully authenticated state**.

* Simply hiding or blocking a page in the frontend is not sufficient.

* This attack does **not** require guessing or stealing the 2FA code.

* The core issue is a flaw in the **authentication flow**.



### Vulnerability



**2FA Simple Bypass**



### Impact



An attacker with a valid username and password may gain unauthorized access to an account without completing the 
required 2FA verification.

