# Inconsistent Security Controls

## Definition

**Inconsistent security controls** are a type of business logic vulnerability that occurs when security rules are not enforced consistently 
across different parts or stages of an application.




A common example is when an application uses a user-controlled attribute, such as an email domain, to determine privileges but fails to 
properly revalidate that attribute when it is changed.



This can allow a normal user to gain access to functionality intended for privileged users.



---



## Lab Objective



Access the administrative panel and delete:


```text


carlos

```



The admin functionality should only be available to employees of:




```text

DontWannaCry

```



---



# Exploitation Concept



The application uses the user's email domain as part of its logic for identifying company employees.



However, the security controls are inconsistent.




We can:



```text

Register a normal account

        ↓

Verify the account

        ↓

Change the email address

        ↓

Use @dontwannacry.com
        ↓

Application treats the account as an employee
        ↓
Access /admin

        ↓

Delete carlos

```



---



# Step-by-Step Exploitation




## 1. Discover the Admin Panel



With Burp Suite running, go to:



```text
Target → Site map

```



Right-click the lab domain and select:




```text

Engagement tools → Discover content

```




Start the content discovery scan.


After a short time, `/admin` should appear in the discovered content.




---




## 2. Access `/admin`



Browse to:



```text

/admin


```


Access is denied.



The response indicates that the admin functionality is only available to:



```text
DontWannaCry

```



This provides an important clue that the application is checking whether the user's account belongs to the company.



---




## 3. Register a Normal Account




Go to the registration page.



Use the email domain provided by the lab's email client:



```text

anything@YOUR-EMAIL-ID.web-security-academy.net

```



The exact `YOUR-EMAIL-ID` value is unique to the lab instance.




---


## 4. Confirm the Account



Open the lab's **Email client**.


Find the registration email and click the confirmation link.



The account is now activated.





---



## 5. Change the Email Address


Log in and go to:



```text

My account

```




The account management functionality allows the email address to be changed.



Change the email address to an address using the company domain:



```text

anything@dontwannacry.com

```




According to the official lab solution, the application accepts this change and subsequently treats the account as belonging to a 

DontWannaCry employee.



---



## 6. Access the Admin Panel




Now browse to:





```text

/admin

```




The account has administrative access.



This demonstrates the inconsistent security control:



```text

Normal account

      ↓

Email changed to company domain

      ↓

Employee status recognized

      ↓

Admin functionality becomes available

```


---




## 7. Delete Carlos



Open the admin panel and delete:



```text

carlos

```


The lab is solved.





---


# Why Did the Exploit Work?


The vulnerability was caused by **inconsistent enforcement of the security rule**.





The application relied on the email domain to determine whether the user was an employee, but the email-change functionality did not properly prevent a normal user from changing their email to the trusted company domain.



Therefore, a user-controlled value influenced authorization:



```text

User-controlled email

        ↓

@dontwannacry.com

        ↓

Employee status

        ↓

Admin access

```




The important point is that we did not directly bypass `/admin`.


Instead, we manipulated a legitimate account-management feature that affected the application's authorization decision.






---



# Root Cause



```text


Inconsistent security controls

            +

Insufficient validation of privilege-related data


            ↓

User changes email domain

            ↓

Application treats user as an employee

            ↓


Unauthorized admin access


```



---



# Key Takeaways



When testing business logic and authorization, check whether security controls are applied consistently across the entire workflow.




Pay particular attention to attributes that may affect privileges:



```text

Email domain



Role

Account type


Organization
Subscription level
User status


```



Ask:


* Can I change an attribute after it has been trusted?

* Is the attribute revalidated after modification?

* Does changing it affect authorization?

* Is the same security rule enforced by every relevant endpoint?

* Can a normal user reach a privileged state through legitimate account functionality?


### Core Lesson



> **Never trust a privilege-related attribute just because it was previously validated. Revalidate it whenever the attribute changes or 

whenever it is used for an authorization decision.**
