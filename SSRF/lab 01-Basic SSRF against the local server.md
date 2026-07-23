# Lab: Basic SSRF against the local server

## Goal
Exploit an SSRF vulnerability to access the local admin panel and delete the user `carlos`.

## Vulnerability
The application accepts a user-controlled URL in the `stockApi` parameter and performs a server-side HTTP request without validating the destination.

## Steps

1. Open any product page.
2. Click **Check stock**.
3. Capture the request in Burp Suite.
4. Send the request to Repeater.
5. Replace the `stockApi` value with:

```text
http://localhost/admin
```

6. Send the request and inspect the response.
7. Find the delete endpoint:

```text
/admin/delete?username=carlos
```

8. Change `stockApi` to:

```text
http://localhost/admin/delete?username=carlos
```

9. Send the request to delete the user.

## Payloads

```text
http://localhost/admin
```

```text
http://localhost/admin/delete?username=carlos
```

## Key Takeaway

The application trusted a user-supplied URL and made a server-side request without validating the destination, allowing access to internal resources such as `localhost`.