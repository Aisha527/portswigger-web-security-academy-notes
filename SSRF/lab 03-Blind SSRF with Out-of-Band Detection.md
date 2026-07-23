# Lab: Blind SSRF with Out-of-Band Detection

## Description

The application uses analytics software that fetches the URL specified in the `Referer` header whenever a product page is visited.

The response is not returned to the attacker, making this a **Blind SSRF** vulnerability.

---

## Vulnerability

The application performs a server-side request using the value of the `Referer` header without validating the destination.

---

## Steps

1. Open Burp Collaborator.
2. Copy the generated Collaborator URL.
3. Visit any product page.
4. Intercept the request.
5. Replace the `Referer` header with your Burp Collaborator URL.

Example:

```http
Referer: https://YOUR-COLLABORATOR-ID.burpcollaborator.net
```

6. Forward the request.
7. Return to Burp Collaborator.
8. Click **Poll now**.
9. Observe an HTTP or DNS interaction.
10. The lab is solved once the interaction is received.

---

## Payload

```http
Referer: https://YOUR-COLLABORATOR-ID.burpcollaborator.net
```

---

## Key Takeaways

- Blind SSRF does not return the server response.
- Burp Collaborator is used for Out-of-Band (OAST) detection.
- A received DNS or HTTP interaction confirms that the server executed the request.
- The `Referer` header can become an SSRF source if the application fetches its value without validation.