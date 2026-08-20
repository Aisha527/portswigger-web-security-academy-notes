# Blind OS Command Injection — Out-of-Band Data Exfiltration


## Definition



**Blind OS Command Injection** occurs when user-controlled input allows OS command execution, but the command 
output is not returned in the HTTP response.



**Out-of-Band (OOB) data exfiltration** uses an external channel, such as DNS, to transfer command output when it 
cannot be retrieved directly.



## Lab



**Lab:** Blind OS command injection with out-of-band data exfiltration

**Difficulty:** Practitioner

**Function:** Feedback



### Goal



Execute `whoami` and exfiltrate the current username through a DNS query to **Burp Collaborator**.



## Why OOB Is Needed



The application:



* Executes the command asynchronously.

* Does not return the command output.

* Does not allow output redirection to an accessible location.

* Allows out-of-band interactions with Burp Collaborator.



Therefore, the HTTP response cannot be used to retrieve the result.




## Exploitation Concept



First, execute:



```bash

whoami

```



The command returns the current username.



Instead of displaying the result in the HTTP response, the username is incorporated into a DNS lookup:



```text

whoami

  ↓
Username

  ↓

DNS query containing the username

  ↓

Burp Collaborator

  ↓

Read the username

```



For example, conceptually:



```text

username.collaborator-domain

```



If the username were `carlos`, the DNS lookup would contain:


```text

carlos.<COLLABORATOR-DOMAIN>

```



The interaction received by Burp Collaborator therefore reveals the command output.



## Lab 4 vs Lab 5



### Lab 4 — OOB Interaction



The goal was only to prove command execution:



```text

OS command

    ↓

DNS lookup

    ↓

Burp Collaborator

    ↓

Interaction detected

```




### Lab 5 — OOB Data Exfiltration



The goal is to transfer the command output:



```text

whoami

    ↓

Username

    ↓

DNS query
    ↓

Burp Collaborator

    ↓
Username retrieved

```



## Attack Flow


```text

Feedback input

      ↓

OS Command Injection

      ↓

whoami

      ↓

Current username

      ↓

DNS-based OOB channel

      ↓

Burp Collaborator

      ↓

Username recovered

```



### Key Takeaway



> **OOB data exfiltration** uses an external interaction channel, such as DNS, to transfer command output when 

direct response-based retrieval and file-based output redirection are unavailable.



**Lab Status:** Not completed — requires Burp Collaborator.


Source: [PortSwigger — Blind OS Command Injection](https://portswigger.net/web-security/os-command-injection/blind?utm_source=chatgpt.com)





