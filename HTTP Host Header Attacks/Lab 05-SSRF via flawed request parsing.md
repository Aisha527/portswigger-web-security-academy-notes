# SSRF via Flawed Request Parsing


**Difficulty:** Practitioner  

**Type:** Server-Side Request Forgery (SSRF)  

**Lab:** PortSwigger Web Security Academy



## 1. Vulnerability Definition



**Server-Side Request Forgery (SSRF)** is a vulnerability that allows an attacker to make a server send requests to unintended 

destinations, potentially including internal services that are not directly accessible from the internet.



In this lab, flawed request parsing allows the attacker to manipulate how the server determines the destination of a request by supplying 

an absolute URL in the request line and a different `Host` header.



## 2. Core Concept




The application validates the absolute URL but handles the `Host` header inconsistently. This discrepancy can be exploited to route 

requests to internal IP addresses.



**Goal:** Access the internal admin panel in `192.168.0.0/24` and delete the user `carlos`.

## 3. Exploitation Steps

### Step 1: Test Host Header Validation

- Send the original `GET /` request to Repeater.

- Modify the `Host` header.

- Observe that the application returns `403 Forbidden`.



**Why?** The application validates the `Host` header when the request uses a relative URL.

### Step 2: Use an Absolute URL

- Change the request line to use the full lab URL:




  `GET https://YOUR-LAB-ID.web-security-academy.net/ HTTP/2`



- Modify the `Host` header and observe the response.



**Why?** The application handles the absolute URL and `Host` header differently, exposing a request-routing flaw.



### Step 3: Enumerate Internal IP Addresses

- Send the request to Burp Intruder.

- Disable **Update Host header to match target**.

- Set the `Host` header to `192.168.0.§0§`.

- Configure the Numbers payload from `0` to `255`, with a step of `1`.

- Compare response status codes and content.

The IP `192.168.0.165` returned a redirect to `/admin`, identifying a promising internal host.

### Step 4: Access the Admin Panel

- Set the `Host` header to `192.168.0.165`.

- Append `/admin` to the absolute URL.

- Send the request and inspect the response.




A `200 OK` response containing the user-deletion form confirmed access to the admin panel.



### Step 5: Delete the User

- Inspect the HTML form to identify:

  - Endpoint: `/admin/delete`

  - Method: `POST`


  - CSRF token

  - Required `username` parameter

- Set the username to `carlos`.

- Include the current CSRF token and valid session cookie.

- Use Burp's **Change request method** option to convert the request to `POST` if needed.

- Send the request and verify that the lab is solved.

## 4. Key Takeaways

- Request parsing inconsistencies can create SSRF vulnerabilities.

- The absolute URL and `Host` header may be interpreted differently by different components.

- A `302` response can indicate a promising route, but the response content must be inspected.


- Internal IP enumeration can help identify exposed administrative interfaces.

- CSRF tokens and valid session cookies may be required to perform administrative actions.

## 5. Mitigation

- Parse and validate request destinations consistently.

- Avoid trusting conflicting destination information from the request line and `Host` header.

- Restrict outbound server requests using allowlists and network-level controls.

- Prevent access to internal administrative services from untrusted request paths.



**Reference:** https://portswigger.net/web-security/host-header/exploiting/lab-host-header-ssrf-via-flawed-request-parsing