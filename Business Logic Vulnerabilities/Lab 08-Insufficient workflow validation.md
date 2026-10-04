# Insufficient workflow validation

**Difficulty:** Practitioner  


**Status:** Solved  

**Category:** Business Logic Vulnerabilities



## Vulnerability



**Insufficient workflow validation** occurs when an application assumes that users will follow a predefined sequence of actions but fails 
to verify that previous workflow steps were actually completed before allowing a later step.



In this lab, the application exposes an order-confirmation endpoint that can be accessed independently of the normal checkout flow.



By replaying the confirmation request after adding an expensive item to the basket, the order can be completed without deducting the item's 
cost from the store credit.



---


## Goal



Buy the:



```text

Lightweight l33t leather jacket

```



without having its cost deducted from the available store credit.



---



## Credentials



```text

Username: wiener

Password: peter

```



---



# 1. Log in



Log in using:



```text

wiener:peter

```



Make sure Burp Suite is running and intercepting the traffic.



---



# 2. Observe a normal checkout



First, purchase any item that you can afford with your store credit.



Open:



```text

Burp Suite → Proxy → HTTP history

```



Study the requests generated during checkout.



One important request is:



```http

POST /cart/checkout

```



This request redirects the user to an order-confirmation page.


---

# 3. Identify the confirmation request



After the checkout request, observe a request similar to:



```http

GET /cart/order-confirmation?order-confirmation=true

```



Send this request to:


```text

Burp Repeater

```




We will replay it later.



---



# 4. Add the leather jacket



Return to the application and add:



```text

Lightweight l33t leather jacket

```



to the basket.


Do not complete the normal checkout flow.



---


# 5. Replay the confirmation request


In Burp Repeater, resend:



```http

GET /cart/order-confirmation?order-confirmation=true

```



The application processes the order successfully.



However, the cost of the leather jacket is **not deducted from the store credit**.



The lab is now solved.



---



## Why does this work?



The intended workflow is approximately:




```text

Add item
   ↓


Checkout

   ↓
Payment / credit deduction

   ↓


Order confirmation

   ↓

Complete order

```



The application incorrectly assumes that reaching the order-confirmation endpoint means that the previous workflow steps were successfully 
completed.




It does not properly verify the current state of the transaction before completing the order.



Therefore, the confirmation endpoint can be replayed independently.



---



## Exploitation Request



```http



GET /cart/order-confirmation?order-confirmation=true

```


### Why this request?


The application treats the confirmation request as sufficient to complete the order instead of verifying that checkout and payment/credit 
deduction have already occurred.



---




# Attack Flow



```text

Login as wiener

      ↓

Purchase an affordable item

      ↓

Observe the checkout flow

      ↓

POST /cart/checkout

      ↓

Redirect to order confirmation

      ↓

GET /cart/order-confirmation?order-confirmation=true

      ↓

Send confirmation request to Burp Repeater
      ↓

Add the leather jacket to the basket

      ↓

Replay the confirmation request

      ↓


Order completed

      ↓

No store credit deducted

      ↓

LAB SOLVED

```



---



# Key Takeaways



- Do not assume that users follow the intended workflow.

- Each sensitive workflow step should validate the application's current state.

- A confirmation endpoint should verify that checkout and payment have actually completed.

- Previous steps should not be inferred from the fact that a later endpoint was reached.

- Workflow state must be enforced server-side.



### Main lesson


> Never trust the sequence of client requests. Validate the server-side state before performing each critical workflow action.