# Web Shell Upload via Extension Blacklist Bypass

### 1. Definition

An **Extension Blacklist Bypass** occurs when an application blocks specific file extensions but fails to 
properly secure the upload process or its server configuration.



In this lab, the vulnerability was exploited by uploading an `.htaccess` file to change how Apache handles 
an otherwise non-executable file extension.



### 2. Steps / Burp Suite




**Step 1: Upload a PHP File**


- Logged in using `wiener:peter`.

- Intercepted the avatar upload request using Burp Suite.

- Uploaded a PHP web shell named `webshell.l33t`.

- Accessed the uploaded file, but PHP code was not executed because `.l33t` was not configured as a PHP 
extension.



**Step 2: Create and Upload the .htaccess File**
- Created a file named `.htaccess`.

- Added the following Apache directive:



  ```apache

  AddType application/x-httpd-php .l33t

  ```



- Modified the `filename` in the upload request:



  ```http

  Content-Disposition: form-data; name="avatar"; filename=".htaccess"

  ```



- Uploaded the `.htaccess` file into the `avatars` directory.






**Step 3: Upload the Web Shell**

- Uploaded the PHP web shell again, using the filename:



  ```http

  filename="webshell.l33t"

  ```



- Kept the PHP code that reads `/home/carlos/secret`.



**Step 4: Execute the Web Shell**

- Accessed the uploaded file:



  ```http

  GET /files/avatars/webshell.l33t

  ```



- Apache applied the `.htaccess` configuration and treated `.l33t` files as PHP.

- The server executed the PHP code and returned the lab secret.



### 3. Why It Works

1. The application used an **Extension Blacklist** to block certain file types.


2. The `.l33t` extension was not included in the blacklist, so the file was accepted.

3. Apache supports directory-level configuration through `.htaccess` files.

4. The `AddType` directive associated the `.l33t` extension with the PHP MIME type.

5. Apache then processed uploaded `.l33t` files as PHP scripts.



### 4. Key Takeaway

**Extension Blacklist Bypass via Server Configuration.**



Relying only on an extension blacklist is insufficient. Allowing users to upload server configuration files 

such as `.htaccess` can change how uploaded files are handled and potentially enable code execution.
