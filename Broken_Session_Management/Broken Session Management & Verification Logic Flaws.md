
# Broken Session Management & Verification Logic Flaws

---

## 1. Password change doesn't invalidate other sessions
**Steps:**
1. Login to your account using Firefox and Chrome.
2. In Firefox, go to settings and change your password.
3. Go back to Chrome and refresh the page.
4. **If you're still logged in → Broken session invalidation.**  
*Expected behavior:* All sessions should be invalidated after password change.

---

## 2. 2FA activation doesn't invalidate other sessions
**Steps:**
1. Login with the same account on Firefox and Chrome.
2. In Firefox, enable 2FA.
3. Go to Chrome and refresh the session.
4. **If you're still logged in without re-authentication → Vulnerability.**

---

## 3. Replayable authenticated request
**Steps:**
1. While logged in, intercept an authenticated request (e.g., update profile) in Burp Suite.
2. Send it to Repeater.
3. Then log out from the app.
4. Try to replay the same request in Repeater (Update anything).
5. **If it's still accepted → Session token still valid = Vulnerability.**

---

## 4. Expired password reset link still works
**Steps:**
1. Request a password reset email, but don't click the link.
2. Log in normally using username & password.
3. Then change your account email/password.
4. Logout.
5. Now, click the original reset link.
6. **If it still works → Critical vulnerability.**

---

## 5. Browser back button shows profile after logout
**Steps:**
1. Logout from your account.
2. Press Alt + Left arrow or the browser Back button.
3. **If it still shows user profile or dashboard data → Broken cache management.**

---

## 6. Email update sends OTP to old email
**Steps:**
1. Try updating your email from `old@example.com` to `new@example.com`.
2. **If OTP is sent to the old email instead of the new one → Logic flaw.**

---

## 7. Email verified status abuse
**Steps:**
1. Create an account with email A → victim.
2. Update it to email B → attacker-controlled, then verify it.
3. After verification, update the email back to email A.
4. **If email A appears as verified without verifying → Vulnerability.**

---

## 8. Verification bypass via email change
**Steps:**
1. Create an account with `victim@gmail.com` (don’t verify it).
2. Update the account to `hacker@gmail.com`.
3. Click the verification link sent to `hacker@gmail.com`.
4. **If the account becomes verified with `victim@gmail.com` → Serious vulnerability.**
