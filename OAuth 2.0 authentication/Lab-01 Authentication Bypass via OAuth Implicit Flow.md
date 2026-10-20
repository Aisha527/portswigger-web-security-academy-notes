# Authentication Bypass via OAuth Implicit Flow

What is OAuth 2.0 Authentication?

OAuth 2.0 is an authorization framework that al

lows applications to access limited user information from another service without requiring users to share their passwords with the client 

application. It is also commonly used as part of social login mechanisms to authenticate users.



# Authentication Bypass via OAuth Implicit Flow




## Vulnerability Definition

An OAuth authentication vulnerability that occurs when the client application trusts user-supplied identity parameters without properly 

verifying that they correspond to the access token. An attacker may exploit this flaw to impersonate another user.


## Lab Goal


Log in to Carlos's account without knowing his password.



**Target email:** `carlos@carlos-montoya.net`


## Exploitation Steps



1. Log in using the social media account credentials: `wiener:peter`.

2. Intercept the `POST /authenticate` request sent after the OAuth callback.

3. Send the request to Burp Repeater.

4. Modify the `email` parameter to Carlos's email address while keeping the other parameters unchanged.



Example request body:



```json

{
  "email": "carlos@carlos-montoya.net",

  "username": "wiener",

  "token": "<current_access_token>"

}

```



5. Send the modified request.

6. Right-click the modified POST request and select **Request in browser → In original session**.

7. Copy the generated URL and visit it in the browser to complete the authentication flow.

8. Verify that you are logged in as Carlos and that the lab is solved.



## Why It Works

The client application fails to properly validate the relationship between the submitted identity information and the access token. This 

allows the attacker to manipulate the email used during authentication and impersonate another user.



## Key Takeaway

Never trust identity parameters submitted by the browser. The server must validate the access token with the OAuth provider and derive the 

authenticated user's identity from trusted, verified data.
