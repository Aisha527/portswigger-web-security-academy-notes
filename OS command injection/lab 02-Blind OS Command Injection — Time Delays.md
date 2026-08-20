# Blind OS Command Injection — Time Delays



## Definition



**Blind OS Command Injection** occurs when user-controlled input allows OS command execution, but the command's 
output is **not returned in the HTTP response**.



Attackers can use side channels, such as **time delays**, to confirm command execution.


## Lab



**Lab:** Blind OS command injection with time delays

**Difficulty:** Practitioner

**Function:** Feedback

**Parameter:** `email`



### Goal



Cause a **10-second delay** using OS command injection.



## Payload



```text

x||ping+-c+10+127.0.0.1||

```



Decoded command:



```bash

ping -c 10 127.0.0.1

```



### Payload Breakdown



* `x` → normal input.

* `||` → shell operator used to inject/chain a command.

* `ping -c 10 127.0.0.1` → produces the observable time delay.

* The final `||` closes the injected portion.



## Why `ping -c 10`?



`-c 10` makes `ping` send 10 requests. In the lab environment, this results in a delay of approximately **10 seconds**.


## How We Confirmed the Injection



The command output is not returned:


```text

Injection

    ↓

Command execution

    ↓
No command output

```



So instead, we observed the response time:


```text

Injection

    ↓

ping command executes

    ↓

~10-second delay

    ↓

Delayed HTTP response

```



The delay confirms that the injected command was executed.



## Lab 1 vs Lab 2



| Lab 1                   | Lab 2                       |

| ----------------------- | --------------------------- |

| Non-blind               | Blind                       |

| Output is returned      | Output is not returned      |

| `whoami`                | Time delay                  |

| Directly observe output | Infer execution from timing |

| Simple case             | Time-based                  |



### Key Takeaway



> **Blind OS Command Injection** can be confirmed through side channels such as time delays when command output is 
not visible.



**Lab solved ✅**


Source: [PortSwigger — OS Command Injection](https://portswigger.net/web-security/os-command-injection?utm_source=chatgpt.com)


