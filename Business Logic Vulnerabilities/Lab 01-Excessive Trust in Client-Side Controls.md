# Business Logic Vulnerabilities


## Lab: Excessive Trust in Client-Side Controls




**Difficulty:** Apprentice

**Category:** Business Logic / Logic Flaw



---



## Vulnerability Overview



**Excessive trust in client-side controls** occurs when an application relies on values or restrictions controlled by the client without 
properly validating them on the server.



Client-side controls should never be treated as a security boundary because an attacker can intercept and modify HTTP requests before they 
reach the server.



In this lab, the application trusted a client-supplied product price.



---



## Lab Objective



Buy the:



**Lightweight "l33t" leather jacket**



for an unintended price.



---



## Login



Provided credentials:



```text

Username: wiener

Password: peter

```



---



## Step 1 — Add the Product to the Cart



After adding the leather jacket to the cart, I intercepted the request using Burp Suite.



The request was:



```http

POST /cart HTTP/2


Host: 0a8900bc0323b5ca80054e0a00f40086.web-security-academy.net


Content-Type: application/x-www-form-urlencoded



productId=1&redir=PRODUCT&quantity=1&price=133700

```



The interesting parameter was:



```text

price=133700

```



The price was being sent directly by the client.



---



## Step 2 — Modify the Price



I changed:



```text

price=133700

```



to:



```text

price=1

```



Modified request body:



```http

productId=1&redir=PRODUCT&quantity=1&price=1

```



I then sent the request.



---



## Step 3 — Complete the Purchase



The application accepted the modified price and added the product to the cart at the manipulated price.



I then completed the checkout process and successfully solved the lab.



---



## Why Did It Work?



The application failed to properly validate the client-supplied `price` on the server.



The application effectively trusted:



```text

price=1

```



instead of independently determining the legitimate price for the product.




### Vulnerable flow



```text

Client

   |

   | productId=1

   | price=1

   v

Server

   |

   | Trusts client-supplied price

   v

Transaction

```




### Secure flow



```text

Client

   |

   | productId=1

   v

Server

   |

   | Retrieves the legitimate price

   | from trusted server-side data

   v

Transaction

```



The vulnerable behavior allowed us to manipulate a **transaction-critical value**.



---



## Key Takeaways



### 1. Never trust client-controlled values



Any value sent by the client can potentially be modified using an intercepting proxy such as Burp Suite.


### 2. Client-side validation is not a security boundary




Restrictions implemented only in the browser can be bypassed by modifying the underlying HTTP request.




### 3. Identify transaction-critical parameters



When testing business logic, look for parameters such as:



```text
price

quantity

discount

productId

userId

amount

```



Then ask:



> Does the server independently validate this value?


qAW2

### 4. Understand the application's workflow



Business logic vulnerabilities are often discovered by understanding how legitimate functionality can be manipulated or combined in 

unintended ways.



---



## Exploitation Summary




```text


1. Login as wiener

2. Add the leather jacket to the cart

3. Intercept POST /cart

4. Identify the client-controlled price parameter

5. Change:





   price=133700


   to:



   price=1



6. Send the modified request

7. Complete the purchase

8. Lab solved


```


---



## Final Payload



```http
productId=1&redir=PRODUCT&quantity=1&price=1

```



## Root Cause



> The server excessively trusted a client-supplied `price` parameter instead of validating the product price server-side.










