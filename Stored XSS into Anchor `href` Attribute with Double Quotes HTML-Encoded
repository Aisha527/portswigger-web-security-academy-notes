# Lab: Stored XSS into Anchor `href` Attribute with Double Quotes HTML-Encoded

## Overview

This lab demonstrates a **Stored Cross-Site Scripting (XSS)** vulnerability where user input is stored inside the `href` attribute of an anchor (`<a>`) element.

Although the application HTML-encodes double quotes (`"`), preventing attribute injection, it still allows dangerous URL schemes such as `javascript:`.

---

## Vulnerability

The application renders user input like this:

```html
<a href="USER_INPUT">Aisha</a>
```

The developer encodes double quotes:

```
"  →  &quot;
```

This prevents an attacker from breaking out of the `href` attribute and injecting a new attribute such as:

```html
" onclick="alert(1)
```

However, the application does **not** validate the value assigned to the `href` attribute.

---

## Exploitation

Instead of escaping the attribute, the attacker supplies a malicious JavaScript URL:

```text
javascript:alert(1)
```

The generated HTML becomes:

```html
<a href="javascript:alert(1)">Aisha</a>
```

When the victim clicks the link, the browser interprets the value as JavaScript and executes:

```javascript
alert(1)
```

---

## Root Cause

The application focuses on encoding quotation marks but fails to validate the URL scheme used inside the `href` attribute.

As a result, dangerous schemes such as `javascript:` are accepted and executed by the browser when the link is clicked.

---

## Key Takeaways

- HTML-encoding double quotes prevents **attribute injection**.
- It does **not** prevent the use of dangerous URL schemes.
- User-controlled values inside `href` attributes should be validated against a whitelist of safe protocols such as `http` and `https`.
- Never rely solely on output encoding for XSS prevention.

---

## Prevention

- Validate URL schemes before inserting user input into `href`.
- Allow only trusted protocols such as `http:` and `https:`.
- Reject dangerous schemes such as `javascript:` and `data:`.
- Apply context-aware output encoding.
- Deploy a strong Content Security Policy (CSP) as an additional layer of defense.
