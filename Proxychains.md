
## ProxyChains: Tool to Route Web Traffic Through Proxies

### What is ProxyChains?
ProxyChains is a Linux tool that routes all traffic coming from any command-line tool through a proxy server. It's the simplest and easiest method to route web traffic of command-line tools through web proxies.

- **Example**:  
  Imagine you're visiting a website directly from your computer. The website sees your real IP address.  
  But when you use ProxyChains, your traffic doesn’t go straight to the website. Instead, it first passes through other computers (called proxies), so the website sees the proxy's IP address, not yours.  
  It's like asking your friend to go to the store for you – the store sees your friend, not you.

#### Why Do We Use ProxyChains?
1. **Hide our real IP address**: Useful for maintaining anonymity.
2. **Bypass firewalls or restrictions**: Helps to access websites if they block you based on IP.
3. **Test website behavior from different locations or IPs**: Essential for penetration testing and verifying geo-restricted content.

**Note**: Proxying tools usually slow them down, so use proxies only when you need to investigate requests, not for regular browsing.

---

### How to Configure ProxyChains

1. **Edit the Configuration File**:  
   Open the file `/etc/proxychains.conf` in a text editor.  
   Comment out the last line (if it exists) and add the following lines to configure the proxy servers:
   ```bash
   socks4 127.0.0.1 9050    # Tor proxy
   http 127.0.0.1 8080      # Local proxy (Burp)
   ```

2. **Check Proxy Connectivity**:  
   To check if the proxies are working, run the following command:
   ```bash
   proxychains curl https://check.torproject.org
   ```
   - If the proxy is working correctly, the output should show the proxy version and the website’s HTML content.
   - You'll also find the request in Burp Suite under Proxy > HTTP History:
     ```
     GET / HTTP/1.1
     Host: check.torproject.org
     User-Agent: curl/7.74.0
     Accept: */*
     Connection: close
     ```

---

### Using ProxyChains with Various Hacking Tools

#### 1. **Opening Browser with Fake IP**  
   To open Firefox through a proxy:
   ```bash
   proxychains firefox
   ```

#### 2. **Downloading Files through Proxy**
   To download files using `wget` or `curl`:
   - **With wget**:  
     ```bash
     proxychains wget http://example.com/file.zip
     ```
   - **With curl**:  
     ```bash
     proxychains curl http://example.com
     ```
   - These commands fetch the page or download the file as if accessed from a different IP.

#### 3. **Using SQLmap**  
   For SQL injection testing with SQLmap, route all traffic through the proxy:
   ```bash
   proxychains sqlmap -u "http://victim.com/page.php?id=*" --dbs
   ```
   - SQLmap will send all traffic through the proxy (fake IP).
   - This is useful if the target website is monitoring IP addresses or has a rate limit.

#### 4. **Using Hydra for Brute-Force Attacks**  
   For brute-forcing a login page using Hydra through the proxy:
   ```bash
   proxychains hydra -l admin -P /path/to/passwords.txt 192.168.1.10 http-post-form "/login.php:username=^USER^&password=^PASS^:Invalid credentials"
   proxychains hydra -l admin -P /path/to/passwords.txt example.com http-post-form "/login.php:username=^USER^&password=^PASS^:Invalid credentials"
   ```

#### 5. **Using Nmap with Proxies**  
   To use Nmap with a proxy, use the `--proxies` flag:
   ```bash
   nmap --proxies http://127.0.0.1:8080 TargetIP -pPORT -Pn -sC
   ```
   - Make sure to check Burp Suite under Proxy > HTTP History to see the requests.

#### 6. **Using Metasploit with Proxies**  
   To route traffic through a proxy in Metasploit:
   - Set the proxy for a specific exploit:
     ```bash
     use auxiliary/scanner/http/robots_txt
     set PROXIES HTTP:127.0.0.1:8080
     set RHOST TargetIP
     set RPORT TargetPort
     run
     ```
   - Again, check Burp Suite under Proxy > HTTP History to monitor the requests.

   **Run Metasploit through ProxyChains**:  
   ```bash
   proxychains msfconsole
   ```
   - This will start Metasploit, routing all traffic through the configured proxy.

---

### Final Notes
- **ProxyChains** is an essential tool for privacy and bypassing restrictions, especially in penetration testing. By routing traffic through proxies, you can ensure that your real IP remains hidden.
- Always ensure that the proxy servers (like Tor or local proxies) are running correctly before using ProxyChains.

**Important**: Proxies can slow down your network speed, so it's best to use them only when needed.
