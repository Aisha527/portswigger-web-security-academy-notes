# Stealing OAuth Access Tokens via a Proxy Page


## 1. Vulnerability Overview



This vulnerability allows an attacker to steal another user's OAuth access token by exploiting insufficient `redirect_uri` validation 
combined with an insecure `postMessage()` implementation in a page within the trusted client application.



The OAuth authorization server trusts the application's domain but does not strictly validate the complete redirect path. As a result, an 
attacker may be able to redirect the OAuth response to a different page on the same domain.



If that page exposes its URL through `postMessage()`, an attacker-controlled parent page can capture the URL containing the access token.



### Key Concepts



- **OAuth Implicit Flow:** Returns the access token directly in the URL fragment.

- **redirect_uri Validation:** Ensures that OAuth responses are redirected only to approved callback URLs.

- **Path Traversal:** Uses `../` to navigate from one path to another.



- **postMessage():** Allows communication between browser windows and frames.

- **Insecure postMessage:** Exposes sensitive information when messages are sent without proper origin restrictions and validation.



## 2. Reconnaissance – Finding the Vulnerable Page



While inspecting the client application, I searched for JavaScript handling communication between browser windows.



I found the following code in:




`/post/comment/comment-form`



```javascript

parent.postMessage(

    {type: 'onload', data: window.location.href},

    '*'

);

```



### Why Is This Vulnerable?




- `window.location.href` returns the complete page URL.

- The URL may contain a fragment.

- In OAuth Implicit Flow, the access token is returned inside the URL fragment.

- `parent.postMessage()` sends this information to the parent window.

- The wildcard (`*`) does not restrict the target origin.



If this page is loaded inside an iframe on an attacker-controlled page, the parent page can receive the message and extract the access 
token.



## 3. Exploiting redirect_uri Validation



I intercepted the OAuth Authorization Request using Burp Suite.



The original request contained:




```http

GET /auth?client_id=CLIENT_ID&redirect_uri=https://LAB-ID.web-security-academy.net/oauth-callback&response_type=token&nonce=NONCE&
scope=openid%20profile%20email

```



The important parameter was:



```text

response_type=token

```



This indicates that the application uses the Implicit Flow, where the access token is returned in the URL fragment.



### Modify redirect_uri



Original callback:



```text

https://LAB-ID.web-security-academy.net/oauth-callback

```




Modified callback:



```text


https://LAB-ID.web-security-academy.net/oauth-callback/../post/comment/comment-form

```



### Why Use ../ ?



The path traversal sequence allows navigation from `/oauth-callback` to `/post/comment/comment-form`, while keeping the same trusted domain.



If the OAuth server does not strictly validate the complete redirect URI, it may accept this modified path and redirect the OAuth response 
to the vulnerable page.





## 4. Setting Up the Exploit Server


I created an exploit page containing:



1. An `iframe` that loads the modified OAuth authorization URL.


2. A `message` event listener to receive messages from the iframe.

3. Logic to extract the received URL.


4. A request that sends the captured data to the Exploit Server.



### Exploitation Logic



The iframe loads the vulnerable comment form.



The page executes:




```javascript

parent.postMessage(

    {type: 'onload', data: window.location.href},

    '*'

);


```



Since the parent window is controlled by the attacker, it can receive the message, extract the URL containing the access token, and send it 
to the Exploit Server.



## 5. Extracting the Access Token





After launching the exploit and making the victim open it, I checked the Exploit Server Access Log.



I found a request containing a URL-encoded OAuth response.



The relevant parameter was:



```text

access_token=STOLEN-TOKEN

```



I decoded the URL-encoded data and extracted the access token.





**Note:** A `404` response in the Access Log does not necessarily indicate that the exploit failed. The request can still reach the Exploit 

Server, exposing the captured data in the logged URL.

## 6. Using the Stolen Token to Access Administrator Data


After obtaining the token, I used Burp Repeater to send a request to the OAuth server's `/me` endpoint.



```http

GET /me HTTP/2


Host: OAUTH-SERVER

Authorization: Bearer STOLEN-ACCESS-TOKEN

```



I replaced the original Bearer token with the stolen access token.





### Analyzing the Response




The server returned the authenticated user's information:




```json

{

  "sub": "administrator",

  "apikey": "ADMIN-API-KEY",

  "name": "Administrator",

  "email": "administrator@normal-user.net",

  "email_verified": true

}

```



The important field was:




```json


"sub": "administrator"
```




This confirmed that the stolen access token belonged to the Administrator account.


I then extracted the administrator's API key and submitted it to complete the lab.



## 7. Why Did the Exploit Work?




The exploit succeeded because multiple weaknesses were combined:



1. The OAuth server did not strictly validate the complete `redirect_uri` path.

2. The application used the Implicit Flow, exposing the access token in the URL fragment.



3. The comment form sent `window.location.href` to its parent using `postMessage()` with a wildcard target origin.

4. The attacker-controlled parent page received the message and extracted the access token.

5. The stolen token could be used to access the `/me` API endpoint and retrieve Administrator data.



## 8. Mitigation



- Enforce exact matching for registered redirect URIs.

- Reject path traversal sequences and unexpected callback paths.


- Prefer Authorization Code Flow with PKCE instead of the Implicit Flow.


- Avoid sending sensitive information, especially access tokens, through `postMessage()`.


- Specify an exact `targetOrigin` instead of using `*`.


- Validate `event.origin` and `event.source` when receiving messages.




**Key Takeaway:**




A page on a trusted domain can still introduce a serious security vulnerability. By combining weak redirect URI validation with insecure 

`postMessage()` handling, an attacker can use a vulnerable page as a proxy to steal OAuth access tokens and access protected API resources.

