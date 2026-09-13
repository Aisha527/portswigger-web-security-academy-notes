# 2FA Bypass Using a Brute-Force Attack 


## 📌 Overview

In this lab, I tackled a multi-factor authentication (MFA/2FA) bypass vulnerability from PortSwigger Web Security 
Academy. The vulnerability arises when a 4-digit verification code (`0000`–`9999`) lacks rate limiting, making it 
theoretically vulnerable to brute-force attacks.



* **Target User:** `carlos:montoya`

* **Vulnerability:** Insecure 2FA Verification / Brute-force Vulnerability

* **Tools Used:** Burp Suite Community Edition & Turbo Intruder Extension



---



## 💡 The Obstacle: Why Standard Brute-Forcing Fails

While a 4-digit code (only 10,000 possibilities) seems trivial to brute-force, the application implements a strict 
session invalidation mechanism:

* Submitting **two consecutive invalid 2FA codes** kills the current session and kicks the client back to the primary `/
login` screen.

* A naive attack (such as standard Intruder or a basic script) quickly fails because subsequent requests hit the generic 
login page rather than the 2FA verification endpoint.



### My Approach

To bypass this limitation, I leveraged Burp Suite's **Session Handling Rules and Macros**. The plan was to automatically 
log in as `carlos` before each 2FA attempt. This guarantees that every brute-force payload is tested against a fresh, 
valid session with an up-to-date CSRF token.



---



## 🔍 Mapping the Authentication Flow

By observing my manual login flow in Burp's Proxy HTTP History, I isolated the four key requests:



1. `GET /login` — Serves the login page and issues the initial CSRF token.

2. `POST /login` — Submits valid credentials (`carlos:montoya`), resulting in a `302 Found` redirect to `/login2`.

3. `GET /login2` — Returns the 2FA submission page containing the input form and a new CSRF token.

4. `POST /login2` — Submits the CSRF token along with the guessed `mfa-code`.



---



## 🛠️ Configuration & Setup



### 1. Configuring the Session Handling Macro

I navigated to **Settings ⚙️ → Sessions → Session Handling Rules → Add**:



* **Scope Tab:**

  * **Tools scope:** Enabled all core tools, paying extra attention to checking **`Extensions`**. *(Crucial step: Since 
  Turbo Intruder operates as an extension, omitting this prevents the macro from running on its traffic).*
  
  * **URL scope:** Selected **`Include all URLs`**.



* **Details Tab:**

  * Added a **`Run a macro`** action.

  * In the macro recorder, I selected the three prerequisite requests in sequence:

    1. `GET /login`

    2. `POST /login`

    3. `GET /login2`

  * Configured parameter handling:

    * ✅ **Update current request with parameters matched from final macro response**: Specified `csrf, session`.

    * ✅ Enabled: `Tolerate URL mismatch when matching parameters`.

    * ✅ **Update current request with cookies from session handling cookie jar**: Selected *Update all cookies except 
    for* (left empty).
  * 
  Ran **Test macro** to verify that the final response returned HTTP 200 with the 4-digit code form intact.



---



### 2. Turbo Intruder Script

After sending the manual `POST /login2` request to Turbo Intruder, I replaced the `mfa-code` parameter value with `%s`:



```http

POST /login2 HTTP/2

Host: 0a2f00a60468c11f8097f8500088002f.web-security-academy.net

...



csrf=awCqR4BOcXDBIhDToV66YPnMvyNUx7h&mfa-code=%s

Then, I configured the following Python script:



Python

def queueRequests(target, wordlists):

    engine = RequestEngine(

        endpoint=target.endpoint,

        concurrentConnections=1,

        requestsPerConnection=1,

        pipeline=False,

        engine=Engine.BURP,

    )



    for i in range(0, 10000):

        code = "{0:04}".format(i)

        engine.queue(target.req, code)





def handleResponse(req, interesting):

    table.add(req)

Why these specific script settings?

engine=Engine.BURP: Bypasses raw socket dispatching and forces requests through Burp's internal networking layer. This is 
required so the Session Handling Rule can trigger the macro before every payload.



concurrentConnections=1 & requestsPerConnection=1: Enforces strict serial request queuing, eliminating race conditions 
between session re-authentication and code submission.



code = '{0:04}'.format(i): Formats numbers with leading zeros (0000 to 9999).



🎯 Exploitation & Root-Cause Troubleshooting

This lab took a fair amount of troubleshooting to get right. Here are the core issues I ran into and how I resolved them:



Issue Encountered	Why It Happened	How I Fixed It

Invalid CSRF token	Turbo Intruder was sending requests without updated CSRF tokens.	Realized I had forgotten to 
enable Extensions in the Tool Scope under Session Handling Rules.

400 Bad Request: Missing parameter 'mfa-code'	Selecting "Update all parameters" caused Burp to wipe the mfa-code 
payload because it wasn't present in the macro's response.	Restricted parameter updates strictly to csrf and session.

Alternating response lengths (3285 vs 3625)	The login page was leaking every second request because sessions expired 
before the macro renewed them.	Switched to serial execution (concurrentConnections=1) and ensured the macro was 
executing on every probe.

Identifying the valid response	Macro internal requests can return 302 Found for /login2.	Checked the Location header 
to ensure it redirected to /my-account?id=carlos, which marked the true positive.

🏆 Lab Completion

Once the attack hit the valid code, the response returned an HTTP 302 Found with the header:



HTTP

Location: /my-account?id=carlos

I right-clicked the winning entry, selected Show response in browser, navigated to the account page, and confirmed the 
bypass was successful!



🔐 Key Takeaways

Never rely on small code spaces for 2FA: A 4-digit code only offers 10,000 combinations, making automated attacks 
feasible even with throttling in place.



Session macro mechanics: Tool scope configuration and single-thread pacing are critical when synchronizing dynamic tokens 
(like CSRF) across rapid requests.



Defense recommendations:



Implement strict per-account rate limiting (e.g., lock out after 3–5 failed MFA attempts).



Increase code length to 6–8 digits with short validity windows (2–3 minutes).