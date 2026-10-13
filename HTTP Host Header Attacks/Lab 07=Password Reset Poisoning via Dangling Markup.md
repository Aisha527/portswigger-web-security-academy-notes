# Password Reset Poisoning via Dangling Markup

## Vulnerability Definition

Password reset poisoning via dangling markup occurs when an application uses untrusted input, such as the `Host` header, to construct 

password-reset email content. An attacker can inject incomplete HTML markup that causes an email scanner to include the remaining email 

content in a request to an attacker-controlled server, potentially exposing a victim's password.



## Root Cause

- The application trusts the `Host` header when constructing links in password-reset emails.


- The email contains HTML content.

- An injected, unclosed HTML attribute can cause subsequent email content to be interpreted as part of an external resource URL.

- An automated email scanner follows or loads the resource, leaking part of the email to the attacker-controlled server.


## Exploitation Steps

1. Submit a password-reset request for the test account `wiener`.

2. Intercept `POST /forgot-password` in Burp Repeater.

3. Modify the `Host` header to inject dangling HTML markup referencing the Exploit Server.

4. Submit the request and inspect the email using **Email client → View raw**.

5. Open **Exploit Server → Access log** and look for a request containing leaked email content.

6. Once the technique is confirmed against the test account, change the username to `carlos` while retaining the working payload.

7. Monitor the Access log for the new request and extract the leaked password.

8. Log in as `carlos` with the recovered password to solve the lab.


## Evidence of Successful Exploitation

The Access log contained a request similar to:


`GET /?/login'>click+here</a>+to+login+with+your+new+password:...`




This indicates that the remaining email content was appended to the resource URL due to the dangling markup.


## Mitigation

- Never use an untrusted `Host` header to generate password-reset links.

- Build reset URLs using a trusted, server-side configured domain.

- Avoid sending passwords by email; use cryptographically secure, short-lived, single-use reset tokens.

- Apply context-aware output encoding and safe HTML templating.

- Ensure email scanners cannot perform sensitive state-changing actions, while maintaining appropriate protections for reset links.



## Key Takeaway

**An untrusted Host header combined with dangling HTML markup can cause automated email scanners to leak sensitive password-reset email 
content.**