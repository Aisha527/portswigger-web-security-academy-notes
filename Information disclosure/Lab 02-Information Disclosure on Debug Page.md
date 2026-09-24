# Information Disclosure on Debug Page


**Difficulty:** Apprentice

**Category:** Information Disclosure

**Status:** Solved ✅



## Overview



**Information Disclosure** occurs when an application exposes internal or sensitive information to users who should not have access to it.



Examples of sensitive information include:



* Environment variables

* Secret keys

* API keys

* Internal paths

* Debug information

* Configuration details

* Stack traces

* Server or framework information



In this lab, a publicly accessible PHP debug page exposed sensitive environment information, including the `SECRET_KEY`.



---



## Lab Objective



Obtain the value of the:



```text

SECRET_KEY

```



environment variable and submit it to solve the lab.



---




## Exploitation



### 1. Browse to the Home Page



With Burp Suite running, browse to the lab's home page.



Then open:



```text

Target → Site map

```



---



### 2. Find the Hidden Debug Link



The home page contains an HTML comment that reveals a hidden debug link.



The comment points to:




```text


/cgi-bin/phpinfo.php

```



This is an important reconnaissance technique because HTML comments can sometimes reveal:



* Hidden endpoints

* Developer notes

* Debug functionality

* Internal paths

* Temporary features



---



### 3. Send the Debug Page to Repeater



In Burp's Site map, locate:



```text

/cgi-bin/phpinfo.php

```



Right-click it and select:



```text

Send to Repeater

```



---



### 4. Request the Debug Page



Send the request from Burp Repeater:



```http

GET /cgi-bin/phpinfo.php HTTP/2

Host: <LAB-HOST>

```



The server responds with a PHP `phpinfo()` page containing detailed debugging and environment information.


---



### 5. Find the Secret Key



Search the response for:



```text

SECRET_KEY

```



The response contained:



```html

<tr>

<td class="e">SECRET_KEY </td>

<td class="v">nzmlu2wyohoveynrbhm5sfd4332581iy </td>

</tr>

```


Therefore, the required value was:



```text

nzmlu2wyohoveynrbhm5sfd4332581iy

```



---



### 6. Submit the Solution



Go back to the lab and select:



```text

Submit solution

```




Submit the value of `SECRET_KEY`.


The lab is then marked as:




```text


Solved ✅

```



---



## Attack Chain



```text

Home Page

    ↓

Hidden HTML Comment

    ↓

Debug Endpoint

    ↓

/cgi-bin/phpinfo.php

    ↓

PHP Debug Information
    ↓

Environment Variables

    ↓

SECRET_KEY

    ↓

Submit Solution

    ↓

Lab Solved
```



---



## Why Did This Work?



The application exposed a PHP debugging page that should not have been publicly accessible.



`phpinfo()` displays detailed information about the PHP runtime and server environment. Because the page was accessible, an attacker could 
retrieve internal configuration and environment variables.



One of those variables was:



```text

SECRET_KEY

```




The vulnerability therefore resulted from exposing **debug functionality and sensitive environment information** to an unauthorized user.



---



## Key Takeaways



* Debug pages should not be exposed in production environments.

* HTML comments can reveal hidden application functionality.

* `phpinfo()` can disclose sensitive server and application information.

* Environment variables may contain secrets and credentials.

* Information disclosure can provide useful information for further attacks.

