## Unicode Normalization Issue
1. victim account: `victim@gmail.com`
2. create an account using Unicode | example: `vićtim@gmail.com` (here is ć is an Unicode character)
   
- list of Unicode character: https://en.wikipedia.org/wiki/List_of_Unicode_characters
Note: check where verification doesn't require

---

## Authorization Issue
1. change email of Account A and put email B
2. check confirmation mail in account B
3. open the confirmation mail from account C (Send the confirmation mail to the victim)
  - Taken over Account C

---

## Reusing Reset Token
- if target allows you to reuse the reset link then hunt for more reset link via `gau`,`wayback` or `urlscan.io`
  - https://urlscan.io/

---


## Pre Account Takeover
1. signup using normal signup form as a hacker but hacker has no verification link.
2. then if victim signs up using oauth.
3. Verification bypass now attacker can login the victim account without verification link with the password he entered while registering.

---

## Host Header Injection
- In this case there are 4 ways do that.
1. click reset password change host header.
2. or change proxy header ex: X-Forwarded-For: attacker.com
3. or change host, referrer, origin headers at once as attacker.com
4. click reset then click resend mail and do all 3 methods above
