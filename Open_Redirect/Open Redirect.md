# Open redirect vulnerability

## What is an open redirect?

- It is a vulnerability that occurs when a website redirects users to a URL specified via a parameter without validating or restricting it.
- This allows an attacker to redirect users to malicious websites using the vulnerable site as a trusted bridge.

### Practical example

Imagine you have a link like this:  
`https://example.com/redirect?url=https://evil.com`  

If the website redirects the user directly to the value of the `url` parameter without checking its safety, then it's vulnerable to Open Redirect.

---

## How is it exploited?

1. The attacker crafts a URL like:  
   `https://example.com/redirect?url=https://phishing.com`
2. The victim clicks the link because it appears to be from a trusted domain (example.com).
3. The website redirects the victim to the malicious site (phishing.com).
4. The victim may enter sensitive information (e.g., credentials), which the attacker then captures.

---

## How can you bypass security filters?

1. `https://example.com/redirect?url=//phishing.com`  
   - Some filters allow `//`, assuming it's a relative path, but it actually leads to an external site.

2. `https://example.com/redirect?url=////phishing.com`  
   - Repeating slashes can bypass poorly implemented filters that only check for one or two slashes.

3. `https://example.com/redirect?url=javascript:confirm()`  
   - Tries to execute JavaScript via the `javascript:` protocol. You may need to encode the colon (`:`), like `javascript%3Aconfirm()`.

4. `https://example.com/redirect?url=www.whitelisted.com.phishing.com`  
   - Tricks filters that check for substrings like `whitelisted.com`, but the full domain is actually a subdomain of `phishing.com`.

5. `https://example.com/redirect?url=//phishing%E3%80%82com`  
   - Uses a Unicode dot instead of a regular dot (`.`) to bypass basic domain validation.

6. `https://example.com/redirect?url=https://phishing%E3%80%82com`  
   - Same as above, but with the `https` protocol included.

7. `https://example.com/redirect?url=//phishing%00.com`  
   - Uses a null byte (`%00`) to try to break the filter or truncate the URL in some systems.

8. `https://example.com/redirect?url=whitelisted.com&url=phishing.com`  
   - Some servers may only read the last value of the repeated `url` parameter, ignoring the first.

9. `http://www.example.com@phishing.com`  
   - Uses the user-info part in the URL to mislead users. Although it looks like `example.com`, it actually goes to `phishing.com`.

---

## Why is it dangerous?

- It can be used in phishing attacks.
- It breaks the trust users have in a legitimate site.
- It allows attackers to mask the real destination of a malicious link.

---

## How to prevent it?

- Filter redirect URLs: Only allow redirects to internal URLs or a trusted whitelist of domains.
- Validate the redirect target: Make sure the redirect URL is safe and expected.
- Avoid passing raw URLs in parameters: Use encrypted values or short-lived tokens instead.
