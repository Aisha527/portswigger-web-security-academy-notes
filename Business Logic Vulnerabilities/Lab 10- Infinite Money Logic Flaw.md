# Lab: Infinite Money Logic Flaw

**Difficulty:** Practitioner  

**Category:** Business Logic Vulnerabilities  

**Status:** Solved ✅



## Vulnerability



The application contains a business logic flaw in its purchasing workflow.



A Gift Card can be purchased for less than its actual value by applying a coupon. The resulting Gift Card can then be redeemed for its full 
value.


By automating this workflow and repeating it, the account balance can be increased until it is sufficient to purchase the **Lightweight 
l33t leather jacket**.



---


## Goal



Buy:



```text

Lightweight l33t leather jacket

```



using the increased account balance.



---



## Credentials


```text

Username: wiener

Password: peter


```



---



## 1. Add a Gift Card to the Cart



Add a Gift Card to the shopping cart.




Request:



```http

POST /cart

```


### Why?



This adds the Gift Card to the current shopping cart.



---



## 2. Apply the Coupon



Apply:



```text
SIGNUP30

```



Request:



```http

POST /cart/coupon

```



### Why?



The coupon reduces the purchase price of the Gift Card.



The important business logic condition is:



```text

Gift Card Value > Purchase Price

```



This creates a positive value difference that can be exploited.



---



## 3. Purchase the Gift Card




Complete the checkout process.



Request:



```http

POST /cart/checkout

```




The application then requests:



```http

GET /cart/order-confirmation?order-confirmed=true

```



The completed purchase results in a new Gift Card code.



---



## 4. Gift Card Code



Each new Gift Card purchase generates a new code.



For example:



```text

Purchase 1 → CODE_1


Purchase 2 → CODE_2


Purchase 3 → CODE_3
```



Therefore, the code cannot simply be hardcoded for every iteration.




The current Gift Card code must be extracted from the current purchase workflow.



---



## 5. Redeem the Gift Card



The Gift Card is redeemed using:



```http

POST /gift-card

```



Example:

```http


POST /gift-card


gift-card=NEW_CODE

```


### Why?



This adds the Gift Card's value to the user's account balance.



---



## 6. Automating the Workflow with Burp Macro



The complete workflow is:



```text

POST /cart

        ↓

POST /cart/coupon
        ↓

POST /cart/checkout

        ↓

GET /cart/order-confirmation

        ↓

Extract the new Gift Card code



        ↓

POST /gift-card

```


A Burp Macro and Session Handling Rule are used to automate this workflow.



### Why?





Every iteration creates a **new Gift Card code**.



The Macro allows Burp to:



1. Purchase a new Gift Card.

2. Obtain its newly generated code.

3. Pass that code to `POST /gift-card`.

4. Repeat the process.


---


## 7. Dynamic Parameter Handling




The Gift Card code from the response is extracted and supplied to the Gift Card redemption request.




Conceptually:


```text

Purchase response

        ↓
Extract Gift Card code

        ↓

POST /gift-card

        ↓

Redeem extracted code

```



This ensures that each iteration uses the correct code generated during that iteration.



---



## 8. Repeat the Workflow



The workflow is repeated multiple times.



Each iteration effectively performs:



```text

Buy Gift Card below its value
        ↓


Receive a new Gift Card code

        ↓

Redeem the full value


        ↓

Increase account balance

```



For example:



```text

Iteration 1 → +$10

Iteration 2 → +$10

Iteration 3 → +$10
...

```



The process continues until the account balance is sufficient to buy the target item.



---



## 9. Verify the Balance



Use:



```http

GET /my-account


```



to verify that the account balance has increased.



---




## 10. Buy the Jacket



Once the balance is sufficient:


```text

Shop

→ Lightweight l33t leather jacket

→ Add to cart

→ Checkout
→ Place order

```



The lab is then solved.



---



# Important Requests



```http

POST /cart

```


Adds the Gift Card to the cart.


```http

POST /cart/coupon

```



Applies the coupon.




```http
POST /cart/checkout

```




Completes the Gift Card purchase.



```http
GET /cart/order-confirmation?order-confirmed=true

```



Confirms the order and provides the current purchase's Gift Card information.





```http

POST /gift-card

```


Redeems the Gift Card and increases the account balance.



```http

GET /my-account

```



Verifies the increased balance.




---



# Key Takeaway




The vulnerability is not simply about reusing one Gift Card code.



The actual flaw is in the **purchasing workflow**:



```text

Purchase Gift Card below its value

        ↓
Redeem its full value

        ↓

Repeat

```



The attacker can repeatedly generate positive account value by combining legitimate application functions in an unintended way.



### Security Lesson



When testing business logic, don't only test individual endpoints.



Ask:



> **Can legitimate operations be combined or repeated in a way that produces an unintended result?**



In this lab:




```text


Discounted Gift Card

        +

Full-value redemption

        +

Workflow automation

        =

Unlimited account credit



```

**Result: Lab Solved ✅**
