
# Rate limit – practical scenarios

## 1. No rate limit on login page
Allowing unlimited login attempts without throttling or CAPTCHA can lead to brute-force attacks.  
Try sending multiple login requests with different passwords and observe the response.

## 2. No rate limit on old password check (change password)
When changing your password, most systems ask you to enter your **current password** before setting a new one.  
Try intercepting the password change request and perform **fuzzing or brute-force** on the `current password` field.  
If the application does not enforce rate limiting or blocking after multiple wrong attempts, an attacker can brute-force the old password and take over the account.

## 3. No rate limit on sending reset password link
An attacker can abuse the reset password feature to spam a victim’s email.  
Try sending many password reset requests for the same account to see if there’s any throttling.

## 4. No rate limit on OTP or 2FA – account takeover
If there’s no rate limit on OTP/2FA verification, an attacker can brute-force the code.  
Use tools like Burp Intruder to test if multiple wrong codes are accepted without temporary block or delay.

## 5. No rate limit on contact us page
Spammers can abuse the contact form to flood the site or admins with messages.  
Try submitting the form repeatedly and see if there's any delay or block.

## 6. No rate limit on comments
Users can spam the comments section by submitting many messages quickly.  
Test by sending repeated comment submissions and observe if there’s any protection.

## 7. No rate limit on reports of comments
A malicious user could mass-report a post/comment to abuse moderation systems.  
Try reporting the same content multiple times and see if the server stops or limits you.

## 8. No rate limit on port 22
If SSH (port 22) is exposed and there's no rate limiting, it's vulnerable to brute-force attacks.  
Use tools like Hydra or Medusa to test multiple SSH login attempts.
