# Lab: Basic SSRF against another back-end system

## Description

This lab contains an SSRF vulnerability in the stock check functionality. The application fetches stock information from an internal back-end server. The goal is to discover the internal admin interface by scanning the `192.168.0.X` network on port `8080`, then delete the user `carlos`.

---

## Vulnerability

The application accepts a user-controlled URL in the `stockApi` parameter and performs a server-side HTTP request without validating the destination.

---

## Steps

1. Open any product page.
2. Click **Check stock**.
3. Capture the request using Burp Suite.
4. Send the request to **Intruder**.
5. Set the last octet of the IP address as the payload position.

Example:

```http
stockApi=http://192.168.0.§1§:8080/admin
```

6. Configure the payload numbers from:

```text
1-255
```

7. Start the attack.
8. Look for the response that differs from the others (usually returns **HTTP 200** with a different response length).
9. The valid IP in this lab was:

```text
http://192.168.0.7:8080/admin
```

10. Open the request in Repeater and verify that the admin interface is returned.
11. Replace the URL with:

```text
http://192.168.0.7:8080/admin/delete?username=carlos
```

12. Send the request to delete the user.

---

## Payloads

### Scan the internal network

```text
http://192.168.0.§1§:8080/admin
```

### Delete the user

```text
http://192.168.0.7:8080/admin/delete?username=carlos
```

> **Note:** The IP address changes each time the lab is generated. Replace `192.168.0.7` with the IP address you discover during your scan.

---

## Key Takeaways

- SSRF can be used to access internal network resources.
- Private IP ranges such as `192.168.x.x` are common SSRF targets.
- Burp Intruder is useful for discovering internal hosts by fuzzing IP addresses.
- Focus on responses with different status codes or response lengths, as they often indicate interesting endpoints.
- After identifying the correct host, use Repeater to interact with the internal service.