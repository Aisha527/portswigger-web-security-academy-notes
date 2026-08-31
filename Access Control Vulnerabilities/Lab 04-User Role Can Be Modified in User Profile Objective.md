# Lab: User Role Can Be Modified in User Profile

## Objective


Access the admin panel and use it to delete the user `carlos`.


### Credentials




```text id="z2y8km"

Username: wiener

Password: peter

```



---



## Vulnerability



The application has an admin panel at:



```text id="4r7x2q"

/admin

```



Access is restricted to users with:



```text id="6m9p1v"

roleid = 2

```



However, the application allows users to modify their own `roleid` through the user profile.



A normal user can change:



```text id="h3k7wd"

roleid=1

```




to:



```text id="p8s4qa"

roleid=2

```



and gain administrator privileges.



---



## Steps to Solve



### 1. Log In



Use the provided credentials:



```text id="f6n2tc"

wiener:peter

```



---



### 2. Modify the User Profile



Go to **My Account / Profile**.



Change a profile value and save the changes to generate a profile update request.



---



### 3. Inspect the Request



Open:



```text id="q1v5zr"
Burp Suite → Proxy → HTTP history

```



Find the request responsible for updating the profile.



Look for the `roleid` parameter.



Example:



```text id="x8k3mp"

roleid=1

```



---



### 4. Change the Role



Modify the parameter:



```text id="a6j9ws"

roleid=1

```




to:



```text id="r2c7vy"

roleid=2

```



Send the modified request.



The application accepts the new role.



---



### 5. Access the Admin Panel



Navigate to:



```text id="m4p8qn"

/admin



```



The admin panel should now be accessible.



---



### 6. Delete Carlos



From the admin panel:




```text id="s7w2kd"


carlos → Delete

```



The lab is solved.



---



## Why Does the Vulnerability Exist?



The application allows the user to modify a parameter that controls their own authorization level.



The server does not properly verify whether the user is authorized to change their `roleid`.



```text id="v5n8qx"


Normal User

     ↓

roleid=1

     ↓

Modify Request

     ↓

roleid=2

     ↓

Administrator

```



---




## Privilege Escalation



This is an example of **Vertical Privilege Escalation**:



```text id="c9m4tx"

Normal User → Administrator

```



The user moves from a lower privilege level to a higher privilege level.



---



## Key Takeaways



* Never allow users to modify their own authorization level.

* Sensitive authorization parameters must be protected server-side.

* Do not trust client-controlled request parameters.

* Always validate authorization before changing roles or privileges.

* Parameter tampering can lead to privilege escalation.


### Vulnerability Type


**User Role Can Be Modified in User Profile**



### Related Concepts



* Access Control

* Broken Access Control

* Role-Based Access Control (RBAC)

* Parameter Tampering

* Client-Controlled Data

* Vertical Privilege Escalation

* Server-Side Authorization

