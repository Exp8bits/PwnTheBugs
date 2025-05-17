# Login Vulnerability Scenarios

## HTTP not HTTPS  
- Issue: If the website uses HTTP instead of HTTPS, all data (like email, password, OTP) is sent in plain text and can be intercepted.  
- Example: If you're on public Wi-Fi and register on a website that uses HTTP, an attacker using tools like Wireshark can sniff your email and password easily.

---

## No rate limit over password  
- Meaning: You can try an unlimited number of passwords without the server blocking or limiting your attempts (Brute-force attack).  
- Practical example: Try logging in with the same username but multiple different passwords (e.g., admin + 100 password attempts). If the site doesn't block or slow you down, it's vulnerable.  
- Fix: The app should implement rate limiting or temporary blocking after a number of failed attempts.

---

## Try default credentials  
- Meaning: Some systems come with default credentials that users often forget to change.  
- Try these common combos:
  ```
  test:test  
  admin:admin  
  admin:password  
  kali:kali  
  admin:123  
  admin:default  
  root:root  
  root:toor  
  admin:kali  
  kali:root  
  admin:123456789
  ```
- Or check the technology the website is using and search for the default credentials for that technology. If any of them work, it's a serious security flaw.

---

## Sql injection in username  
- Meaning: Try injecting SQL code in the username field.  
- Example:  
  - Username: `admin' OR 1=1; -- -`  
  - Password: anything  
- If it lets you log in without correct credentials, it's vulnerable to SQL injection.

---

## Response manipulation to bypass login  
- Meaning: Use tools like Burp Suite to modify the HTTP response from the server to trick the client into thinking you're logged in.  
- Practical step: Send a login request with wrong credentials. Intercept and change the response:  
  - From: `{"success": false, "isAdmin": false}`  
  - To: `{"success": true, "isAdmin": true}`  
- If it redirects you as logged in, the app isn't validating sessions securely.

---

## Send request to SQLMAP  
- Meaning: Take the login request and test it using SQLMAP to detect SQL injection.  
- Example:  
  - `sqlmap -r login_request.txt --level=5 --risk=3 --batch`  
  - `login_request.txt` should contain the raw HTTP request (exported from Burp or browser dev tools).

---

## Inject XSS payloads in username  
- Meaning: Insert XSS code in the username field and see if it's executed.  
- Example: `<svg/onload=confirm()>`  
- If a popup appears (confirm box), the app is vulnerable to XSS.

---

## Template injection  
- Meaning: Some apps use template engines (like Jinja2). If code inside input fields is rendered and executed, it's vulnerable.  
- Example: `{{7*7}}`  
- If the page shows `49`, the app is vulnerable to server-side template injection (SSTI).

---

## View source code (CTRL+U)  
- Meaning: Check the HTML source code of the page (press Ctrl + U) to see if any sensitive data is exposed.  
- Practical check: Look for things like:  
  - `value="admin"`  
  - `password="123456"`  
  - `api_key="secret"`  
- These are sensitive values and should never be in client-side code.

---

## From login (JSON format) to LFI  
- Example:  
  - `{"username":"\`cat /etc/passwd\`", "password":"test"}`  
  - `{"username":"\`touch /var/www/html/hacked.html\`", "password":"test"}`  

---
