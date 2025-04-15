# HTML Injection & XSS Basics

## Where should I inject my payload?

1. **Input Fields**
   - Such as comment fields, search bars, or login forms.

2. **URL Parameters**
   - Example: `http://example.com/page?param=<script>alert('XSS')</script>`

3. **HTTP Headers**
   - Some applications reflect headers without encoding, making them vulnerable.

4. **User-Agent or Referer**
   - If a website reflects User-Agent or Referer values directly into the response, these can be injection points.

5. **Cookies**
   - If the application displays cookie values in HTML without encoding, you can inject there.

6. **File Upload**
   - If the website allows uploading files containing HTML or JavaScript code.
   - Uploading files containing HTML or JavaScript can be dangerous if the content is rendered directly.

---

## ▪ How to Know If the Vulnerability Exists?

- Start by injecting simple HTML elements:
  - Example: `<h1>test</h1>` or `<u>test</u>`
- If the website renders the code instead of treating it as text, it is **vulnerable to HTML Injection**.

---

## ▪ Exploitation Examples

- `<img src=x onerror=confirm()>`
- `<iframe src="http://evil.com"></iframe>`
- `<a href="https://www.evil.com">Click here</a>`

> **Note:** Once you confirm HTML Injection, always try to escalate to **XSS**. If escalation is not possible, report it as **HTML Injection** only.

---

## How to Prevent This as a Developer?

- **In PHP:** Use `htmlspecialchars()` to encode user input.
- **In JavaScript:** Use sanitization libraries like `DOMPurify`.
- **Best Practice:** Always **sanitize and encode** any user input before displaying it in HTML.

