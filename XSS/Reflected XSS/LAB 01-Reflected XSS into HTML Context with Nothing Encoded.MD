# Lab: Reflected XSS into HTML Context with Nothing Encoded


**Difficulty:** Apprentice

**Vulnerability:** Reflected XSS

**Injection Point:** Search functionality

**Context:** HTML context



## Methodology



1. Entered special HTML characters such as `<` and `>` into the search box.

2. Checked the HTTP response to see whether these characters were encoded.

3. The characters were reflected **without encoding**.

4. Inspected the response and found the input reflected inside a `<div>` element.

5. Since the input was in an HTML context and HTML special characters were not encoded, HTML markup could be injected.

6. Used a JavaScript execution payload to verify the vulnerability.

7. The `alert()` function executed successfully, confirming the XSS.

8. The lab was solved.



## Key Takeaways



* **Reflection alone does not mean XSS.**

* Always identify the **context** where the input is reflected.

* Test how special characters are handled before choosing a payload.

* In an HTML context, check whether characters such as `<` and `>` are encoded.

* If HTML characters are reflected as-is, injected HTML may be interpreted by the browser.

* `alert()` can be used as a simple proof of JavaScript execution in PortSwigger labs.



## Thinking Process



**Input → Reflection → Context → Encoding/Filtering → Can I change the parsing? → Can I achieve JavaScript execution?
**



## Result



**Solved ✅**

