#  Insufficient Session Expiration

##  What is Insufficient session expiration?
- A session must end once the **password or email is changed** or **2FA is added**.
- If you're logged into the same account on multiple devices and you change any sensitive information (like password or email), you **should be logged out of all sessions**.
- If you're **not logged out from other sessions**, this is a vulnerability known as **Insufficient Session Expiration**.

---

##  How to find the vulnerability
- Log into the **same account from two browsers**.
- Change the password on one browser.
- Check if the other browser is **still logged in**.

###  Secure behavior:
- All sessions are **logged out automatically**.

###  Vulnerable behavior:
- Other sessions are **still active** — this means **Improper Session Expiration**.

---

##  Impact of Insufficient Session Expiration
- If sessions remain active after sensitive updates:
  - An **attacker who has access** to one session will stay logged in, even after you change your password.
  - Same applies to adding **2FA** — if older sessions are still working, **they bypass the protection**.

---

##  How developers should fix it
- The backend **must invalidate all active sessions** after sensitive changes.
- Use:
  - `Session.invalidate()`
  - **Token Rotation**
  - **JWT Revocation** on password change

---

##  Example vulnerable & secure flask implementation

```python
from flask import Flask, session, redirect, request, url_for

app = Flask(__name__)
app.secret_key = "secret123"

# Fake user database
users = {"user": {"password": "123", "sessions": []}}

@app.route("/login", methods=["POST"])
def login():
    username = request.form["username"]
    password = request.form["password"]
    if users.get(username) and users[username]["password"] == password:
        session["username"] = username
        # Store session ID
        sid = request.cookies.get(app.session_cookie_name)
        users[username]["sessions"].append(sid)
        return "Logged in"
    return "Login failed"

@app.route("/change_password", methods=["POST"])
def change_password():
    username = session.get("username")
    if not username:
        return "Unauthorized", 401
    new_password = request.form["new_password"]
    users[username]["password"] = new_password
    # ⚠ Vulnerable: does not clear other sessions
    return "Password changed"

@app.route("/secure_change_password", methods=["POST"])
def secure_change_password():
    username = session.get("username")
    if not username:
        return "Unauthorized", 401
    new_password = request.form["new_password"]
    users[username]["password"] = new_password
    #  Secure: clear all sessions after password change
    users[username]["sessions"] = []
    session.clear()
    return "Password changed and sessions invalidated"
```

---

##  Summary
--------------------------------------------------------------------------------------------------
| Behavior                                  | Secure? | Explanation                              |
|-------------------------------------------|---------|------------------------------------------|
| Session invalidated after password change |   Yes   |   Protects against session hijacking     |
| Old sessions still active                 |   No    |   Leaves account vulnerable to attackers |
--------------------------------------------------------------------------------------------------
