# OAuth Account Hijacking via redirect_uri

## 1. Vulnerability Overview



### What is OAuth?



OAuth 2.0 is an authorization framework that allows an application to obtain authorization from another service. In the login flow used in 

this lab, the application redirects the user to an OAuth server and receives an authorization code after the authorization step.






### What is redirect_uri?





The `redirect_uri` parameter specifies where the OAuth server sends the user after authorization.



The legitimate callback in this lab was:



`https://LAB-ID.web-security-academy.net/oauth-callback`



The OAuth server should validate this URI against an exact allowlist of registered callback URLs.



If the validation is weak, an attacker may be able to change the URI to an attacker-controlled server and capture the authorization code.



### Impact





If an attacker steals a victim's usable authorization code, they may be able to complete the OAuth login flow as that victim.



In this lab, the victim is the administrator. The final objective is to access the admin panel and delete user `carlos`.



---



## 2. Reconnaissance: Inspecting the Normal OAuth Flow


1. Opened the lab and selected **My account → Log in with social media**.

2. Used Burp Suite → Proxy → HTTP history to inspect the requests.

3. Found the `/social-login` request.

4. Inspected its HTML response and identified the OAuth authorization URL in the meta refresh element.

5. Identified the following parameters:


   - `client_id`: identifies the application.

   - `redirect_uri`: specifies the callback destination.

   - `response_type=code`: requests an authorization code.

   - `scope=openid profile email`: requests the listed identity-related scopes.



The authorization request followed this general structure:



`/auth?client_id=...&redirect_uri=...&response_type=code&scope=openid%20profile%20email`



The OAuth server returned `302 Found`, with a `Location` header pointing to the application's callback and including a `code` parameter.



The browser then requested `/oauth-callback?code=...`, and the application confirmed that social login had succeeded.



This established the normal OAuth flow before testing the redirect URI validation.



---



## 3. Testing redirect_uri Validation


Sent the authorization request to Burp Repeater and modified `redirect_uri` to point to the Exploit Server.



The OAuth server accepted the modified URI and returned a redirect similar to:



`HTTP/2 302 Found`



`Location: https://EXPLOIT-SERVER/?code=...`



This showed that the authorization response could be redirected to an attacker-controlled destination.



A `302` response alone does not prove that the entire account takeover has succeeded. The important detail is the destination in the 
`Location` header and whether the authorization code reaches the controlled server.



---



## 4. Preparing the Exploit Server



1. Opened the lab's Exploit Server.

2. Created an iframe that loaded the OAuth authorization URL.

3. Changed `redirect_uri` from the legitimate lab callback to the Exploit Server's base URL.

4. Saved the page using **Store**.



The iframe was intended to make the victim's browser initiate the OAuth authorization request when the victim visited the exploit page.



The initial attempts did not work as expected:


- Using the original callback sent the code back to the lab.

- Requests to stale `/interaction/...` paths returned `SessionNotFound: invalid_request`.



The interaction endpoint depends on a valid OAuth interaction state and should not be treated as a standalone authorization request.



---



## 5. Delivering the Exploit and Capturing the Code



Selected **Deliver exploit to victim**, then inspected the Exploit Server's Access log.



The first request looked like:


`GET /exploit/`



This only confirmed that the victim loaded the exploit page.



A subsequent request looked like:



`GET /?code=STOLEN-CODE`



The request used a Victim user-agent, showing that the authorization code had reached the Exploit Server through the victim's browser.



This was the key evidence that the code had been leaked.



However, code leakage alone did not mean the administrator account had already been taken over.



---



## 6. Completing the OAuth Flow



Initially, submitting the stolen code to the lab callback resulted in an internal server error. A later message stated:



`Client Error: You already have a logged in session on the site`



The current browser session was still logged in as `wiener`.



To complete the lab:

1. Logged out of the blog application.


2. Copied the latest authorization code from the victim's request in the Exploit Server Access log.

3. Navigated to:


   `https://LAB-ID.web-security-academy.net/oauth-callback?code=STOLEN-CODE`



4. The OAuth flow completed, and the session became authenticated as the administrator.




The important lesson was that the stolen authorization code had to be used to complete the OAuth flow, and the existing `wiener` session 
had to be cleared first.



---



## 7. Completing the Lab



1. Opened the admin panel at `/admin`.

2. Deleted user `carlos`.

3. Confirmed that the lab was marked as solved.


---



## 8. Key Takeaways



- Inspect the normal OAuth flow before modifying requests.

- Check how the application validates `redirect_uri`.

- Inspect the `Location` header in OAuth redirects.


- Distinguish between loading the exploit page and leaking the authorization code.

- Use the Access log to identify the code delivered by the victim.


- A leaked authorization code does not automatically mean the account takeover is complete.

- Check the current authentication session before reusing the stolen code.

- Complete the callback flow and verify which account is authenticated.



## 9. Remediation






- Validate `redirect_uri` using exact matching against pre-registered callback URLs.

- Avoid unsafe prefix or substring validation.

- Do not redirect authorization responses to untrusted destinations.


- Follow secure OAuth authorization-code flow practices and protect authorization codes from leakage.

