# Forced OAuth Profile Linking



## Vulnerability Definition

Forced OAuth profile linking occurs when an application fails to properly protect the account-linking process against CSRF. An attacker can 
trick a victim's browser into linking the attacker's OAuth account to the victim's application account.



## Steps to Solve

1. Log in to the blog as `wiener:peter`.

2. Click **Attach a social profile** and authenticate using the OAuth credentials `peter.wiener:hotdog`.

3. Verify that logging in through the linked social profile works, then log out.

4. Start Burp Proxy interception and initiate the account-linking process again.

5. Intercept the callback request:



   ```http

   GET /oauth-linking?code=STOLEN-CODE

   ```



6. Copy the full request URL and **drop the request** to prevent the authorization code from being consumed.

7. Log out and open the Exploit Server.

8. Put the following payload in the **Body** section, replacing the placeholders with the actual lab URL and captured code:



   ```html

   <iframe src="https://YOUR-LAB-ID.web-security-academy.net/oauth-linking?code=STOLEN-CODE"></iframe>

   ```



9. Click **Store** and then **Deliver exploit to victim**.

10. Return to the lab and click **Log in with social media**.

11. Access the admin panel and delete the user `carlos`.



## Why It Works


- The application does not properly validate the account-linking request against CSRF.

- The authorization code identifies the attacker's OAuth account.

- The victim's browser sends the callback request in the context of the victim's authenticated application session.

- The application links the attacker's OAuth identity to the victim's account.

- The attacker can then authenticate through the linked OAuth profile and gain access to the victim's account.



## Key Takeaway

**OAuth account linking must be protected against CSRF.** Use a cryptographically random `state` value, bind it to the user's session, and 

validate it during the callback. Also ensure authorization codes are short-lived and single-use.



## Important Note

The Exploit Server access log records requests made to the Exploit Server itself. The iframe's callback request targets the lab domain, so 

it will not necessarily appear in the Exploit Server access log.

