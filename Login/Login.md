# Login Vulnerability Scenarios

## HTTP not HTTPS  
- **Issue:** If the website uses HTTP instead of HTTPS, all data (like email, password, OTP) is sent in plain text and can be intercepted.  
- **Example:** If you're on public Wi-Fi and register on a website that uses HTTP, an attacker using tools like Wireshark can sniff your email and password easily.

---

## No rate-limit over password  
- **Meaning:** You can try an unlimited number of passwords without the server blocking or limiting your attempts (Brute-force attack).  
- **Practical example:** Try logging in with the same username but multiple different passwords (e.g., admin + 100 password attempts). If the site doesn't block or slow you down, it's vulnerable.  
- **Fix:** The app should implement rate limiting or temporary blocking after a number of failed attempts.

---

## Try default credentials  
- **Meaning:** Some systems come with default credentials that users often forget to change.  
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

## SQL Injection in username  
- **Meaning:** Try injecting SQL code in the username field.  
- **Example:**  
  - Username: `admin' OR 1=1; -- -`  
  - Password: `anything`
- If it lets you log in without correct credentials, it's vulnerable to SQL injection.

---

## Response manipulation to bypass login  
- **Meaning:** Use tools like Burp Suite to modify the HTTP response from the server to trick the client into thinking you're logged in.  
- **Practical step:** Send a login request with wrong credentials. Intercept and change the response:  
  - From: `{"success": false, "isAdmin": false}`  
  - To: `{"success": true, "isAdmin": true}`  
- If it redirects you as logged in, the app isn't validating sessions securely.

---

## Send request to SQLMAP  
- **Meaning:** Take the login request and test it using SQLMAP to detect SQL injection.  
- **Example:** 
  - `sqlmap -r login_request.txt --level=5 --risk=3 --batch`  
  - `login_request.txt` should contain the raw HTTP request (exported from Burp or browser dev tools).

---

## Inject XSS payloads in username  
- **Meaning:** Insert XSS code in the username field and see if it's executed.  
- **Example:** `<svg/onload=confirm()>`  
- If a popup appears (confirm box), the app is vulnerable to XSS.

---

## Template injection  
- **Meaning:** Some apps use template engines (like Jinja2). If code inside input fields is rendered and executed, it's vulnerable.  
- **Example:** `{{7*7}}`  
- If the page shows `49`, the app is vulnerable to server-side template injection (SSTI).

---

## View source code (CTRL+U)  
- **Meaning:** Check the HTML source code of the page (press Ctrl + U) to see if any sensitive data is exposed.  
- **Practical check:** Look for things like:  
  - `value="admin"`  
  - `password="123456"`  
  - `api_key="secret"`  
- These are sensitive values and should never be in client-side code.

---

## From login (JSON format) to LFI  
- **Example:**
  ```
    {"username":"\`cat /etc/passwd\`", "password":"test"}
  ```
  ```
   `{"username":"\`touch /var/www/html/hacked.html\`", "password":"test"}`  
  ```
---

## Check for logic flaws in login flow
- **What to check:**  
  Some apps have logic mistakes in the login process. Try to bypass authentication by skipping steps or messing with requests.  
- **Examples:**  
  - Access `/dashboard` or `/admin` directly before logging in.  
  - Use Burp Suite to change the server's response after a failed login (like changing `"success": false` to `"success": true"`).  
- **Why:**  
  If the backend doesn’t properly validate, you might get in without valid credentials.

---

## Manipulate JWT tokens
- **What to check:**  
  JWT (JSON Web Tokens) are used to store session info. Test if you can change them and still be accepted.  
- **Examples:**  
  - Change the `alg` from `HS256` to `none`.  
  - Modify claims like `"admin": false` to `"admin": true`.  
  - Use tools like `jwt-cracker` or `hashcat` to crack weak signatures.  
- **Tools:**  
  `jwt-tool`, `jwt-cracker`, Burp’s JWT editor.  
- **Why:**  
  If the app doesn't verify tokens properly, you can become an admin or access data you shouldn't.

---

## Check for session fixation
- **What to check:**  
  See if you can set a session ID before logging in, and if the server keeps using it after login.  
- **Practical step:**  
  - Visit the login page.  
  - Set your own cookie (e.g., `PHPSESSID=1234`).  
  - Log in.  
  - If the session ID is still the same, it’s vulnerable.  
- **Why:**  
  Attackers can hijack sessions by setting the session ID for a victim.
> Send the victim a link with a specific session ID (e.g., in an email or SMS).
> When the victim logs in, the server continues to use that same session ID.
> The attacker can then use the same session ID to access the victim's account.

---

## Bruteforce token-based login (PIN, OTP, etc.)
- **What to check:**  
  Apps that use short PINs (like 4-6 digits) can be cracked if there's no rate limiting.  
- **How:**  
  - Use Burp Intruder or `ffuf` to try all possible PINs (`0000-9999`).  
  - If there's no slowdown or blocking, it’s vulnerable.  
- **Why:**  
  Attackers can guess the code and log in as another user.

---

## Check for caching of sensitive pages
- **What to check:**  
  Login pages and user dashboards shouldn’t be cached in the browser or by proxies.  
- **Practical step:**  
  - Look at response headers like:  
    - `Cache-Control: no-store`  
    - `Pragma: no-cache`  
  - If these headers are missing, sensitive data might be stored and accessible to others.  
- **Why:**  
  On shared computers, cached pages can leak sensitive info.

---

## Username enumeration
- **What to check:**  
  Some apps reveal if a username or email exists based on error messages.  
- **How:**  
  - Log in with an invalid username and see what error you get.  
  - Log in with a real username and wrong password, then compare messages.  
- **Why:**  
  If the messages are different, attackers can confirm valid usernames.

---

## Subdomain or endpoint-based auth bypass
- **What to check:**  
  Some apps have different login paths or subdomains that might not be secured the same way.  
- **How:**  
  - Send login requests to `/api/v2/auth` or other endpoints instead of the main login page.  
  - Look for old endpoints that still work.  
- **Why:**  
  If these endpoints don’t check auth properly, you can log in without real credentials.

---

## Forceful browsing + IDOR after login
- **What to check:**  
  After logging in, try to access data from other users by changing IDs in URLs or parameters.  
- **Examples:**  
  - Visit `/user/profile?id=1337` to see another profile.  
  - Open files like `/invoice/17.pdf`.  
- **Why:**  
  If you can see or download things you shouldn't, the app is vulnerable to IDOR (Insecure Direct Object References).
