# Lab: Stored XSS into HTML Context with Nothing Encoded


**Difficulty:** Apprentice

**Vulnerability:** Stored XSS

**Injection Point:** Comment functionality

**Context:** HTML context



## Lab Description



The application contains a **stored cross-site scripting vulnerability** in the comment functionality.



The goal is to submit a comment that executes JavaScript when the blog post is viewed.



## Methodology



1. Opened the blog post and located the **comment functionality**.

2. Submitted a simple test input to determine whether the comment was reflected.

3. Tested special HTML characters such as `<` and `>` to check whether they were encoded.


4. The characters were reflected **without HTML encoding**.

5. Inspected the response and identified that the comment was placed inside an **HTML context**.

6. Since the input was stored and HTML characters were not encoded, HTML markup could be injected into the comment.

7. Submitted a JavaScript execution payload using the `alert()` function.

8. The payload was stored as part of the comment.

9. When the blog post was viewed again, the stored payload was rendered by the browser and JavaScript executed.

10. The successful execution of `alert()` confirmed the **Stored XSS vulnerability**.

11. The lab was solved.



## Why Is It Stored XSS?



The important difference is that the malicious input is **stored by the application** and executed later when the 

page containing the comment is viewed.



### Request Flow





```text

Attacker Input

      ↓

Comment Submission

      ↓

Server-Side Storage

      ↓

Blog Post Viewed

      ↓

Stored Comment Rendered

      ↓

JavaScript Execution

```



## Context



The comment was reflected in an:



**HTML context**



The application did not properly encode HTML special characters before rendering the stored comment.



## Key Takeaways



* **Stored XSS** occurs when malicious input is stored by the application and later rendered to users.

* The input does not have to be reflected immediately in the same response.

* Always identify where user-controlled input is stored and where it is later rendered.

* Test how special characters such as `<` and `>` are handled.

* **HTML context + insufficient output encoding** can lead to XSS.

* `alert()` is useful as a simple proof of JavaScript execution in PortSwigger labs.

* Stored XSS can potentially affect multiple users who view the affected content.



## Reflected vs Stored XSS



```text

Reflected XSS:

Input → Request → Response → Execution


Stored XSS:

Input → Request → Storage → Later Response → Execution

```


The main difference is that **Stored XSS persists after the original request**, while Reflected XSS is reflected 
directly in the response.


## Testing Mindset



When testing a possible XSS injection point:



```text

1. Find the input

2. Check whether it is reflected/stored

3. Identify the context

4. Test special characters
5. Check encoding/filtering

6. Determine whether the input can alter the parsing context

7. Test for JavaScript execution

```



## Result



**Solved ✅**
