### Race Condition

#### What is a Race Condition?
- A **Race Condition** occurs when two or more operations (processes or threads) run simultaneously and compete to access the same resource (such as a file, variable, or database). If these operations are not properly synchronized, it can lead to unexpected or incorrect behavior.

##### Simple example:
- Imagine you and your brother are both logged into the same bank account from different phones:
  - You try to withdraw $100.
  - Your brother also tries to withdraw $100 at the exact same time.
  - The account only has $150.

**What should happen** is: when one withdrawal begins, the system should block the other until it finishes.

- **But if there's no proper synchronization**, this may happen:

1. Both apps read the balance: $150.
2. Both say "Great! I can withdraw."
3. Both withdraw at the same time.
4. The final balance becomes **-50$**.

✔ That’s a Race Condition.

---

#### Security Example

Imagine a website with a feature to transfer points or money:

1. You click the “Transfer” button.
2. The server checks if you have enough balance.
3. Then it performs the transfer.

✔ If you click the button twice quickly (or send two requests at the same time), both checks might happen **before the balance is updated** — allowing you to transfer more than you should.

---

### Race Condition Scenario

A user has **100 points**. On the "Transfer Points" page, a typical request looks like this:

```http
POST /transfer HTTP/1.1
Host: vulnerable-app.com
Content-Type: application/x-www-form-urlencoded
Cookie: session=abc123

amount=50&to_user=ahmed
```
- Exploiting this scenario using burp intruder

1. Open **Burp Suite** and turn on Intercept.
2. Trigger a point transfer from your browser.
3. Send the request to **Intruder**.
4. Mark the `amount` value (or leave it fixed).
5. Choose an attack type like **Sniper** or **Cluster Bomb**.
6. The idea is to send **multiple requests very quickly** to exploit the race window.

- If you send the same request **10 times at once**:
  - The server might check your balance **before any update** is made.
  - Each request sees that you have enough points.
  - All requests get processed!
**Result**: You end up transferring 500 points instead of just 50 — even though you only had 100.

### Alternative Using Burp Repeater

- Open 5 Repeater tabs for the same request.
- Group them and run them in **parallel**.
- Send them all simultaneously.

---

### Using burp Turbo Intruder

- Send the request to **Turbo Intruder**.
- Choose the `race-single-packet-attack.py` script.
- Launch the attack.

---

### Where to try Race Condition attacks

- **Likes**: A post allows only one like, but sending 5 like requests at the same time results in 5 likes.
- **Coupons**: A 20% discount coupon applied 5 times simultaneously results in a 100% discount.
- **Invitations**: A group limited to 5 members receives 10 because the request was sent twice in parallel.
- **Limitations**: A service limited to 5 uses gets used 10 times due to parallel requests.
- **Payments**: A $100 payment request is sent 5 times simultaneously and charges the account $500.

---

### How to prevent Race Conditions

- Use **locks or mutexes** to ensure only one operation can access a shared resource at a time.
- Use **database transactions** to handle multi-step operations as a single atomic unit.
- Implement **rate limiting** and **proper server-side synchronization**.
