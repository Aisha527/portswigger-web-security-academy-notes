# Remote Code Execution via Web Shell Upload

### Definition

An unrestricted file upload vulnerability occurs when an application allows users to upload files without properly validating them. If an 
attacker can upload a server-side script to a directory where the server executes it, this may lead to Remote Code Execution (RCE).


### Request / Payload

- Intercepted the avatar upload request using Burp Suite.

- Changed the uploaded filename extension from `.jpg` to `.php`.

- Created a PHP file that reads the lab's secret file:

  ```php

  <?php

  echo file_get_contents('/home/carlos/secret');

  ?>

  ```

- Uploaded the PHP file and accessed its URL.


### Why It Works

1. The application did not properly restrict uploaded file extensions.

2. The uploaded PHP file was stored in a publicly accessible directory.

3. The server was configured to execute PHP files in that directory.

4. Accessing the uploaded file caused the server to execute the PHP code and return the secret.



### Key Takeaway

Successful file upload does not automatically mean code execution. RCE requires the uploaded script to be accessible and executed by the 
server.