# Web Shell Upload via Content-Type Restriction Bypass



### 1. Definition



A file upload vulnerability occurs when an application relies on the user-controllable `Content-Type` header to validate uploaded files 

instead of properly checking their actual content.



### 2. Request / Payload

- Intercepted the image upload request using **Burp Suite**.


- Identified the `Content-Type` header inside the file part.

- Changed the file's `Content-Type` from `application/x-php` to `image/jpeg`, while keeping the `.php` extension.

- Forwarded the modified request.

- The application accepted and uploaded the PHP file.

- Accessed the uploaded file through its URL to retrieve the lab secret.



### 3. Why It Works

1. The application relied on the file's `Content-Type` header for validation.

2. This header is controlled by the client and can be modified.

3. Changing it to `image/jpeg` bypassed the application's file type check.

4. The server stored the PHP file in a publicly accessible directory where PHP execution was enabled.

5. Accessing the uploaded file caused the server to execute the PHP code.



### 4. Key Takeaway

**Never trust the Content-Type header alone.**




Client-supplied headers are not reliable proof of a file's actual type. Applications should perform proper server-side validation.




**Important:** Bypassing Content-Type validation does not automatically mean RCE is possible. The uploaded file must also be accessible and 
executable by the server.
