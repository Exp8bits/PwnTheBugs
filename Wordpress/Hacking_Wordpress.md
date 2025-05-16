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
  - Visit `https://target.com/feed` or `https://target.com/?feed=rss2`
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
### Add user to the server
- Search for `Authorization` in javascript files and request to `POST /wp-json/wp/v2/users` to add user.

  ```bash
  POST /wp-json/wp/v2/users
  Authorization: Bearer <authorization_token_you_found_in_js_files>`
  ```
- body of the request:
  ```bash
	{
	"username":"yourUsername",
	"email":"yourEmail",
	"password":"yourPassword",
	"roles":["administrator"]
	}
  ```
- Navigate to /wp-login.php/ and login.

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

### Manual Recon

#### Check `readme.html`, `license.txt`
- Paths:
  - `https://target.com/readme.html` ➝ shows WP version
  - `https://target.com/license.txt`

#### Check `.git`, Backup Files
```bash
https://target.com/.git/
https://target.com/wp-config.php.bak
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
