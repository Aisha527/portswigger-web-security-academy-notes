# Authentication bypass via flawed state machine


**Difficulty:** Practitioner  

**Status:** Solved  

**Category:** Business Logic Vulnerabilities / Authentication



## Vulnerability




A **flawed authentication state machine** occurs when an application assumes that users will progress through authentication states in the 
intended order without properly validating the current authentication state.



In this lab, users must select their role after logging in before reaching the home page.






However, if the role-selection step is skipped, the application falls back to the `administrator` role.



This allows an authenticated low-privileged user to bypass the intended role-selection workflow and access the admin interface.



---



## Goal



Bypass the authentication workflow, access:



```text id="7q5m4x"

/admin

```



and delete:

```text id="2v9k1c"

carlos

```


---


## Credentials



```text id="z4n8lp"

Username: wiener

Password: peter

```



---


# 1. Complete a normal login



Log in using:


```text id="q1v6ds"

wiener:peter

```


After authentication, observe that the application does not immediately take you to the home page.



Instead, you are redirected to:




```text id="w8k3ja"


/role-selector

```



The user is expected to select a role before continuing.



---



# 2. Discover the admin panel





Use Burp's **Content Discovery** functionality to identify hidden application paths.


The admin panel is available at:

```text id="f5r2cx"

/admin

```



Trying to access `/admin` while still on the role-selection page does not work.


---


# 3. Intercept the login flow


Log out and return to the login page.








Turn on:



```text id="n9c5vk"

Proxy → Intercept → ON

```



Log in again with:


```text id="k4s7qa"

wiener:peter


```



Burp captures:



```http id="d2m8xf"


POST /login


```



Forward this request.



---


# 4. Identify the role-selection state


The next request is:




```http id="b6x3wp"

GET /role-selector

```


This request represents the transition to the role-selection state.

---


# 5. Drop the role-selector request


Instead of forwarding:



```http id="s9r2hd"



GET /role-selector

```


click:


```text id="v3m7ka"

Drop

```




This prevents the application from entering the role-selection step.




---



# 6. Access the home page



Now browse directly to the lab's home page.


The application considers the login flow complete even though the role-selection state was skipped.



The user's role defaults to:


```text id="c8f1mz"

administrator


```


You can now access:



```text id="a2q7nv"

/admin

```


---


# 7. Delete Carlos



Open the admin panel:



```text id="j6w4rp"

/admin

```



Find:



```text id="p3k8vd"


carlos

```



and click **Delete**.



The lab is now solved.




---



# Key Exploitation Request



The critical request is:




```http id="m7x2qa"


GET /role-selector

```




Instead of forwarding it, we drop it.


### Why?



The application assumes that the user will necessarily complete the role-selection state after authentication.



By dropping the request, we skip this state:





```text id="u5c9fz"
Login

   ↓

Role Selector

   ↓

Select role

   ↓
Home

```



and instead force:



```text id="q8v3lr"


Login

   ↓
[Role Selector skipped]

   ↓

Home

```



The application then falls back to the insecure default role:



```text id="y4n6pc"

administrator

```



---



# Attack Flow




```text id="b8m2ks"


Login as wiener

       ↓

POST /login

       ↓


Forward

       ↓

GET /role-selector

       ↓

DROP
       ↓

Browse to home page

       ↓

Default role = administrator


       ↓

Access /admin

       ↓

Delete carlos

       ↓



LAB SOLVED

```



---



# Key Takeaways



- Authentication should be implemented as a properly validated server-side state machine.

- The server must not assume that users followed the expected sequence of authentication steps.

- Every privileged state transition should be explicitly validated.

- Default roles must never grant unintended administrative privileges.

- Skipping a workflow step should not result in privilege escalation.



### Main Lesson

> Never trust the sequence of client requests to determine authentication state. Validate the current authentication state and 
authorization requirements on the server before granting access.
