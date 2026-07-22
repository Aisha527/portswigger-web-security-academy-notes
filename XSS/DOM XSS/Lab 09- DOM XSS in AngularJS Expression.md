# Lab: DOM XSS in AngularJS Expression with Angle Brackets and Double Quotes HTML-Encoded

## Idea

The developer properly sanitizes the input by:

- HTML-encoding `<` and `>`.
- HTML-encoding double quotes (`"`).

This means:

- We cannot inject a new HTML tag.
- We cannot break out of an attribute or a value.

However, the application uses **AngularJS**, which gives us another attack vector.

---

## Solution

The page contains the `ng-app` directive, which tells AngularJS to manage that part of the page.

Anything inside:

```text
{{ ... }}
```

is treated as an **AngularJS expression** and evaluated by the framework.

Our first attempt might be:

```javascript
{{alert(1)}}
```

Unfortunately, this doesn't work.

**Why?**

Because AngularJS does not allow direct access to `window.alert` inside expressions. Instead, it looks for `alert` in the current AngularJS scope.

---

## Exploiting AngularJS

We can use the built-in `$on` function, which already exists in the AngularJS scope.

Since `$on` is a JavaScript function, it has a `constructor` property that points to the native `Function` constructor.

### Payload

```text
{{$on.constructor('alert(1)')()}}
```

---

## Payload Breakdown

### `$on`

A built-in AngularJS function used to listen for events.

### `constructor`

Every JavaScript function has a `constructor` property that references the `Function` constructor.

```javascript
$on.constructor('alert(1)')
```

is equivalent to:

```javascript
Function('alert(1)')
```

This creates a new JavaScript function containing our payload.

Finally,

```javascript
()
```

executes the function, resulting in:

```javascript
alert(1)
```

---

## Why Single Quotes?

The application HTML-encodes double quotes (`"`), so we use single quotes instead:

```javascript
'alert(1)'
```

instead of:

```javascript
"alert(1)"
```

---

## Final Payload

```text
{{$on.constructor('alert(1)')()}}
```

---

## Key Takeaways

- `<` and `>` are HTML-encoded, preventing HTML injection.
- Double quotes are also HTML-encoded, preventing attribute injection.
- AngularJS expressions (`{{ }}`) are still evaluated.
- The `$on.constructor()` technique allows us to execute arbitrary JavaScript and trigger DOM XSS.

---

## References

- PortSwigger Web Security Academy – DOM-based XSS  
  https://portswigger.net/web-security/cross-site-scripting/dom-based

- PortSwigger Research – DOM-based AngularJS Sandbox Escapes  
  https://portswigger.net/research/dom-based-angularjs-sandbox-escapes