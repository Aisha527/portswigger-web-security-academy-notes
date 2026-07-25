Description

The application provides a stock check feature that fetches data from an internal system. A blacklist-based filter blocks specific keywords to prevent SSRF attacks.

Objective

Access the admin interface and delete the user carlos.

Protection

The application uses a blacklist to block dangerous values such as:

localhost
127.0.0.1
admin

Instead of validating the destination, it simply searches for blocked keywords in the URL.

Exploitation
Bypass the blocked host

Use an alternative representation of the loopback address:

127.1

instead of

127.0.0.1
Bypass the blocked path

Double URL encode the letter a in /admin:

admin
↓
%61dmin
↓
%2561dmin
Final Payload
http://127.1/%2561dmin/delete?username=carlos
Why it works
127.1 resolves to 127.0.0.1, bypassing the blacklist.
%2561 is decoded twice by the server, resulting in the character a.
The blacklist checks the raw input before decoding, allowing the payload to pass.
Key Takeaways
Blacklists are weak because the same value can have multiple valid representations.
Alternative IP formats can bypass hostname filters.
URL Encoding and Double URL Encoding can bypass keyword-based filtering.