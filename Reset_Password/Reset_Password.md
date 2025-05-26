# Password reset security testing checklist

## 1. Use of HTTP instead of HTTPS
- **Issue:** Reset password links sent via HTTP instead of HTTPS.
- **Risk:** Anyone on the same network can sniff the reset token.
- **Example:** Reset link like `http://example.com/reset?token=xyz123`.

---

## 2. Token leakage in request or response
- **Issue:** Reset token appears in request or response headers/bodies.
- **Risk:** Attackers can capture the token and use it maliciously.
- **Test:** Use Burp Suite to send reset requests and inspect if tokens are visible in responses or headers.

---

## 3. Token reuse (insufficient expiration)
- **Issue:** Reset tokens remain valid after first use.
- **Risk:** Attackers can reuse tokens multiple times to reset passwords.
- **Test:** Try opening the same reset link multiple times; if it still works, token invalidation is missing.

---

## 4. Host header injection in reset link
- **Issue:** Reset links are generated based on the Host header.
- **Risk:** Attackers can modify `Host header` or `X-Forwarded-For` or `X-Forwarded-Host` to generate malicious reset URLs pointing to attacker-controlled domains.
- **Test:** Modify `Host header` or `X-Forwarded-For` header or `X-Forwarded-Host` header in requests and check if reset links in emails reflect the modified domain.

---

## 5. Email bombing due to no rate limiting
- **Issue:** No rate limiting on password reset requests.
- **Risk:** Attackers can flood victim’s inbox with reset emails (email bombing).
- **Test:** Use Burp Intruder to send many reset requests for the same email and observe if rate limiting or blocking occurs.
> Set the payload to `NULL` and choose the number of requests you want to send for example, set the payload count to 1000. If there's no rate limiting on the password reset form, the victim will receive 1000 reset password emails from the company.

---

## 6. Old reset links work after password change
- **Issue:** Reset tokens are still valid after user changes password.
- **Risk:** Attackers can use old tokens to reset passwords even after legitimate password changes.
- **Test:** Request reset link, don't use it, login and change your password, then try old reset link.

---

## 7. Reset link works after email address change
- **Issue:** Reset tokens remain valid even if user changes their email.
- **Risk:** Password reset functionality bypass due to stale tokens.
- **Test:** Request reset password, don't use it, login and change email, then try old reset link.

---

## 8. Active sessions not terminated after password change
- **Issue:** Changing password does not invalidate other active sessions.
- **Risk:** Attackers logged in on other devices remain authenticated after password change.
- **Test:** Log in from multiple devices, change password on one, check if others remain logged in.

---

## 9. OTP brute-force vulnerability
- **Issue:** No rate limiting or protection on OTP verification attempts.
- **Risk:** Attackers can brute-force OTP codes.
- **Test:** Try guessing OTPs rapidly and check if attempts are blocked or limited.

---

## 10. Cross-site scripting (XSS) in reset link
- **Issue:** Reset link parameters vulnerable to XSS.
- **Risk:** Attackers can inject malicious scripts to steal cookies or perform actions.
- **Test:** Inject payloads like `<svg onload=alert(document.cookie)>` into reset link parameters.

---

## 11. SQL Injection in reset password request
- **Issue:** Reset password endpoint vulnerable to SQL injection.
- **Risk:** Database manipulation or data leakage.
- **Test:** Use tools like SQLMap `sqlmap -r request.txt` on password reset requests.

---

## 12. Weak password policy enforcement during reset
- **Issue:** System allows weak passwords on reset.
- **Risk:** Accounts vulnerable to guessing or brute-force.
- **Test:** Try setting weak passwords (e.g., `123456`, `password`) during reset.

---

## 13. UUID v1 tokens vulnerable to sandwich attack
- **Issue:** Use of UUID v1 for reset tokens which contain timestamp.
- **Risk:** Attackers can predict or manipulate tokens.
- **Test:** Check token format and test for timestamp manipulation.
> an attacker could exploit it to perform a sandwich attack by modifying the timestamp embedded in the token.

---

## 14. Reset password link exposed via referrer header
- **Issue:** Reset token sent in HTTP Referrer header.
- **Risk:** Token leak when user clicks external links.
- **Test:** While performing a password reset, click on any social media icon like Facebook or Gmail. Intercept the request and check if the reset token appears in the Referrer header

---

## 15. Sensitive information exposure in response during reset
- **Issue:** Response reveals sensitive victim information (email, username).
- **Risk:** Information disclosure aiding attackers.
- **Test:** Inspect responses for unintended data leaks.

---

## 16. Predictable tokens based on email similarity
- **Issue:** Tokens generated in a way that tokens for similar emails resemble each other.
- **Risk:** Attackers can predict or forge tokens.
- **Test:** Compare tokens from similar email addresses for patterns.
> Request password reset for: `victim@gmail.com` ➜ token = `abcdef123456`
> Request password reset for: `xvictim@gmail.com` ➜ token = `abcdef123460`

---

## 17. Email address manipulation in reset requests
- **Issue:** System does not sanitize or validate email input properly, allowing injection of multiple or malformed emails.
- **Risk:** Can lead to unintended behavior or information leaks.
- **Test:** Send reset requests with multiple or manipulated email fields.
```json
{"email":"victim@mail.com","email":"attacker@mail.com"}
{"email":"attacker@mail.com\nvictim@mail.com"}
{"email":"Victim@mail.com,attacker@mail.com","email":"Victim@mail.com"}
{"email":"Victim@mail.com","email":"Victim@mail.com%0Abcc:Attacker@mail.com"}
{"email":"Victim@mail.com","email":"Victim@mail.com,Attacker@mail.com"}
{"email":"Victim@mail.com;Attacker@mail.com","email":"Victim@mail.com"}
{"email":"\"attacker\"@mail.com"}
{"email":"attacker@mail.com\nCc:admin@company.com"}
{"email":"victim@mail.com\r\nBcc:attacker@mail.com"}
{"email":"victim@mail.com%0D%0ABcc:attacker@mail.com"}
{"email":"victim@mail.com%0aContent-Type:text/html%0a%0a<script>alert(1)</script>"}
{"email":"victim@mail.com","email":"victim+test@mail.com"}
```

---

## 18. Response manipulation to expose victim's email
- **Issue:** Attacker can manipulate request/response to reveal victim’s email.
- **Risk:** Information leakage.
- **Test:** Try changing parameters or IDs in responses to access other users’ data `id=1000 ➜ id=1001`.

---

## 19. IDOR vulnerability in password reset
- **Issue:** Password reset requests contain email or identifier parameters that can be changed to reset others’ passwords.
- **Risk:** Unauthorized password resets.
- **Test:** Modify email parameter in reset requests and verify if system allows changing others’ passwords.

---

## 20. Exposure of sensitive identifiers in headers
- **Issue:** Sensitive IDs (user ID, session ID) appear in HTTP headers during reset.
- **Risk:** Attackers may exploit these for unauthorized access.
- **Test:** Inspect HTTP headers for exposed identifiers `X-User-ID` or `X-Session-ID`.

---

## 21. Cookie hijacking during password reset
- **Issue:** Attacker uses victim’s cookies to hijack session and reset password.
- **Risk:** Unauthorized account takeover.
- **Test:** Attempt resetting password with victim’s cookies, verify session validation.
> You can steal the victim's cookies through XSS.

---

## 22. Vulnerable API endpoint for password reset
- **Issue:** Reset password API endpoint can be manipulated to reset any user’s password.
- **Risk:** Unauthorized resets.
- **Test:** Try modifying email/user parameters in API requests.
> `/api/user/resetpassword?email=test@example.com` ➜ `/api/user/resetpassword?email=victim@example.com`

---

## 23. Open redirect via callback parameters
- **Issue:** Password reset requests include callback URLs vulnerable to open redirect.
- **Risk:** Phishing or redirecting victims to malicious sites.
- **Test:** Analyze the request of reset password and search for callback parameter.
> `callback=https://example.com` ➜ `callback=https://evil.com` or `https:\\evil.com`

---

## 24. Open Redirect in password reset
- **Issue:** The application allows a user to specify a `redirect` or `next` parameter in the password reset URL. This parameter is not properly validated, which allows an attacker to redirect users to a malicious website.
- **Risk:** An attacker can craft a link like `https://target.com/reset?redirect=https://attacker.com`. If a user clicks it, they will be redirected to the attacker's site after completing the password reset process. This can be used for phishing attacks or to capture sensitive information.
- **Test:** Change the `redirect` parameter to a site you control.
> **From**  `https://target.com/reset?redirect=https://target.com/dashboard` **to**  `https://target.com/reset?redirect=https://attacker.com`

---

# Additional recommendations

- **Use HTTPS exclusively** for all password reset flows.
- **Implement strong rate limiting** on reset requests and OTP verification.
- **Invalidate tokens immediately after use** or expiration.
- **Invalidate all active sessions after password change.**
- **Enforce strong password policies** on resets.
- **Sanitize all user inputs** (emails, headers).
- Use security tools like **Burp Suite, Intruder, SQLMap** for thorough testing.
- Monitor logs for suspicious reset activity to detect abuse or attacks.
