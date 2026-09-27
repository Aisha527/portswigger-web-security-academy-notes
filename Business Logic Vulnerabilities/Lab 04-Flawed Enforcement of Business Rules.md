# Flawed Enforcement of Business Rules

## Vulnerability Definition



**Flawed enforcement of business rules** occurs when an application incorrectly implements a business rule, allowing an attacker to bypass 
an intended restriction by manipulating the normal workflow.



In this lab, the application attempts to prevent users from applying the same coupon code multiple times, but the validation only checks 
the previously applied coupon instead of tracking all coupons already used in the order.



## Lab Goal



Buy the **Lightweight l33t leather jacket** despite having insufficient store credit.

## Credentials



```text

Username: wiener

Password: peter

```



## Exploitation Steps



### 1. Log in


Log in using:



```text

wiener:peter

```



After logging in, a coupon code is available:




```text

NEWCUST5

```



### 2. Get the second coupon



Subscribe to the newsletter at the bottom of the page.


This provides another coupon:


```text

SIGNUP30

```



### 3. Add the product




Add the following product to the cart:






```text

Lightweight l33t leather jacket


```



### 4. Apply the coupons





Go to checkout and apply both coupon codes:



```text

NEWCUST5

SIGNUP30

```



Both codes are accepted.





### 5. Bypass the coupon restriction




Trying to apply the same coupon twice consecutively fails:



```text

NEWCUST5

NEWCUST5

```



The second application is rejected because the coupon has already just been applied.



However, alternating between the two coupons bypasses this validation:


```text

NEWCUST5

SIGNUP30

NEWCUST5


SIGNUP30

NEWCUST5
...
```

The application accepts the coupons because it only checks the immediately previous coupon instead of maintaining a list of all coupons 

already used.







### 6. Reduce the order total



Continue alternating between:



```text

NEWCUST5

SIGNUP30


```



until the order total becomes lower than the remaining store credit.



Then complete the order.



## Root Cause


The application incorrectly enforces the coupon-use restriction.



Conceptually, the vulnerable logic behaves like:



```text

if current_coupon == previous_coupon:

    reject

else:


    apply_coupon

```



A secure implementation should track all coupons already used in the current order:


```text

if current_coupon in used_coupons:

    reject

else:

    apply_coupon

```



## Key Takeaway



The important lesson is that business logic vulnerabilities often occur when an application validates only the **current state or immediate 
previous action** instead of enforcing the complete business rule across the entire workflow.



Here, alternating two valid coupons allowed repeated discounts and eventually reduced the purchase price enough to complete the transaction.
