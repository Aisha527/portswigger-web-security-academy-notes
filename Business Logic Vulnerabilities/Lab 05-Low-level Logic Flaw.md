# Low-level Logic Flaw

## Definition



A **low-level logic flaw** occurs when an application fails to properly validate low-level input values or the limits of the underlying 
data types, allowing an attacker to manipulate the application's behavior in an unintended way.



In this lab, the application restricted the `quantity` parameter to two digits per request, but failed to properly handle the cumulative 
price after repeatedly adding large quantities of the same product.



This eventually caused an **integer overflow**, making the cart total wrap around to a negative value.



---



## Lab Goal



Buy the:



```text


Lightweight l33t leather jacket

```



for an unintended price.



Credentials:



```text

Username: wiener

Password: peter

```



---



## 1. Identify the Input Validation



When adding the jacket to the cart, the application sends a request similar to:



```http

POST /cart HTTP/2

Host: <LAB-HOST>

Content-Type: application/x-www-form-urlencoded



productId=<JACKET_ID>&quantity=1&redir=PRODUCT
```



The relevant parameters are:





```text

productId

quantity

redir

```




Testing the `quantity` parameter showed that:



```text

quantity=99

```



was accepted.



However:



```text

quantity=100


```


was rejected with:



```text

Invalid parameter quantity

```



### Observation



The application limits the quantity in each individual request to two digits.



However, it does not properly limit the **cumulative number of items** added through repeated requests.



---



## 2. Send the Request to Intruder



Send the `POST /cart` request to **Burp Intruder**.



Keep:



```text

quantity=99

```




We do not need to fuzz the value itself.



Instead, we want to repeatedly send the same request.



---



## 3. Configure Intruder



### Payload type



```text

Null payloads

```



The payload is empty because we only need to repeat the request.



### Payload configuration


```text

Continue indefinitely

```



This causes Intruder to repeatedly send:


```text

quantity=99

```

---

## 4. Configure the Resource Pool




Create/use a Resource Pool and set:



```text

Maximum concurrent requests = 1

```




This ensures that requests are processed one at a time.




This is important because we need the cart total to increase predictably and want to observe the point at which the integer overflow occurs.



---




## 5. Trigger the Integer Overflow



The jacket costs:


```text

$1,337

```




or:



```text

133700 cents

```



The maximum value of a signed 32-bit integer is:



```text

2,147,483,647

```



When the cumulative cart price exceeds this value, the integer overflows.


Conceptually:



```text

2,147,483,647

        ↓
     overflow
        ↓
-2,147,483,648

```



The cart total therefore wraps around and becomes a large negative value.





---



## 6. Move the Negative Total Toward Zero



Once the overflow occurs, every additional request adds:



```text


99 × 1337

```



to the cart total.


Because the current value is negative, repeatedly adding more jackets moves the total toward zero.



The objective is to stop at a manageable negative value.



---




## 7. Adjust the Total



After reaching a suitable negative value, stop the continuous attack.



The official lab solution then uses a controlled number of requests and manually adjusts the quantity.

For example:



```text

quantity=47

```


can be used in Burp Repeater.




The lab's expected intermediate total is:



```text

-$1221.96

```


---

## 8. Add Another Product



A jacket costs too much to simply add another one without potentially exceeding the `$100` store credit.



Therefore, add a cheaper product and use its quantity to bring the total into the valid range:





```text

$0 < Total < $100

```


The final total can then be paid using the available store credit.



---



## 9. Complete the Lab


Once the cart total is within the available `$100` credit:





```text

Place order



```

The lab should be solved.



---



## Important Requests / Payloads



### Maximum accepted quantity



```text

quantity=99

```


### Rejected quantity


```text

quantity=100

```


### Intruder configuration



```text

Payload type: Null payloads

Payload configuration: Continue indefinitely

Maximum concurrent requests: 1

```


### Manual adjustment



```text
quantity=47

```



Expected intermediate total:

```text

-$1221.96

```

---

## Root Cause



The application validates the quantity of an individual request but fails to properly validate the **cumulative result** of repeated 
requests.


The vulnerable logic can be summarized as:



```text

Per-request validation

        ↓

quantity <= 99

        ↓

Repeated requests
        ↓

Huge cumulative quantity

        ↓

Integer overflow


        ↓


Negative cart total

```



---



## Key Takeaway



Input validation should not only consider whether an individual request contains an acceptable value.



It should also consider:



* The cumulative effect of repeated requests.

* The maximum value supported by the underlying data type.

* The final business result after processing multiple requests.



In this lab, `quantity=99` was individually valid, but repeatedly adding `99` jackets caused the backend integer to overflow and the cart 
price to become negative.


### Vulnerability chain


```text

Insufficient validation


        ↓

Repeated valid requests

        ↓

Integer overflow

        ↓

Negative cart total

        ↓

Price manipulation

```
