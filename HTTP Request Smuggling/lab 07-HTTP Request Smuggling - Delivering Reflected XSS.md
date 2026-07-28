# HTTP Request Smuggling - Delivering Reflected XSS

## Concept


This attack combines **HTTP Request Smuggling (CL.TE)** with a **Reflected XSS** vulnerability.


The attacker smuggles a request that triggers a reflected XSS. Due to request desynchronization, the response 
generated for the smuggled request is delivered to the next user's request instead of the attacker.


As a result, the victim receives a poisoned response containing the XSS payload.



---


## Attack Flow



Attacker Request

        │

        ▼

Front-end (Content-Length)

        │

        ▼

Back-end (Transfer-Encoding)

        │

        ▼

Smuggled Request

(GET /post?postId=...)

        │

        ▼

Server generates a response containing the reflected XSS

        │

        ▼

Victim sends a request

        │

        ▼

Victim receives the poisoned response

        │

        ▼

JavaScript executes in the victim's browser



---


## Payload


```http

POST / HTTP/1.1

Host: target

Content-Type: application/x-www-form-urlencoded

Content-Length: <Calculated by Burp>

Transfer-Encoding: chunked



0



GET /post?postId=4 HTTP/1.1

Host: target

User-Agent: "><script>alert(1)</script>

```



---


## Why does this work?


- The front-end treats the request as a single request.

- The back-end interprets the request differently due to CL.TE desynchronization.

- The smuggled `GET` request is processed first.

- Its response remains in the response queue.

- The next victim receives that response instead of their own.

- Since the application reflects the `User-Agent` header without proper sanitization, the victim executes the 
injected JavaScript.



---



## Key Notes


- Use **HTTP/1.1**, not HTTP/2.

- The vulnerable input is the **User-Agent** header.

- Verify the reflected XSS before attempting request smuggling.

- Unlike the previous lab, there is **no POST body or large inner Content-Length** because the goal is **response 
queue poisoning**, not capturing another user's request.

- Repeat the attack several times until the simulated victim receives the poisoned response.



| Previous Lab                                                            | Current 
Lab                                                              |

| ----------------------------------------------------------------------- | 
------------------------------------------------------------------------ |

| Capture the victim's **request**                                        | Poison the victim's 
**response**                                         |

| Smuggle `POST /post/comment`                                            | Smuggle `GET /post?...
`                                                  |

| Goal: steal the victim's session                                        | Goal: execute reflected XSS in the 
victim's browser                      |

| Increase inner `Content-Length` to capture more of the victim's request | No large inner body needed because the 

attack targets the response queue |
