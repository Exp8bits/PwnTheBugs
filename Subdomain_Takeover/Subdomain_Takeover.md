# Subdomain Takeover Methods

There are two ways to perform a subdomain takeover: manually and automatically using tools.

## Manually:

### 1. Check the subdomain:
- Command: `dig <domain>`
- Command: `dig example.com`
- Command: `dig sub.example.com`
- Command: `dig @8.8.8.8 example.com`
- If the CNAME record doesn't appear, run:
  - Command: `dig example.com CNAME`

### 2. Check CNAME:
- If the subdomain has a CNAME but is not found, it might be unclaimed.
- Try accessing the CNAME, and if it's available, you can buy it and control this subdomain.

## Automatically:

### Case (1): Extracting subdomains and identifying 404s
1. Use `subfinder` to extract subdomains:
   - Command: `subfinder -d example.com | httpx -sc 404 | tee subs.txt`
   
2. Inspect the subdomains with a 404 status code.
3. Pay attention to any unclaimed services (like S3 buckets) or other vulnerabilities.
4. Use the **"Can I Take Over XYZ" GitHub repo** to verify whether the CNAME is vulnerable or not. This repo contains a list of vulnerable CNAME records.

### Case (2): Using Nuclei
- Nuclei is a vulnerability scanner that can automate subdomain takeover detection.
- It supports custom scanning templates, including those for identifying subdomain takeovers.
- Templates for takeovers can be found under the `nuclei-templates/http/takeovers` directory.

Command to run Nuclei:
- Command: `nuclei -l subs.txt -t /path/to/nuclei-templates/http/takeover -o results.txt`

### Case (3): Using Subzy
1. To test a single subdomain:
   - Command: `subzy run --target sub.example.com`
   
2. To test a list of subdomains, hide errors, and check if they are vulnerable:
   - Command: `subzy run --targets subs.txt --hide_fails --vuln`
   
3. To increase the request rate (requests per second):
   - Command: `subzy run --targets subs.txt --hide_fails --vuln --concurrency 100`
   
4. If Akamai's firewall interferes and causes incorrect results:
   - Command: `subzy run --targets subs.txt --hide_fails --vuln | grep -v -E "Akamai|xyz|available|\-"`

## Verifying vulnerability:
Once you find a vulnerable subdomain, use the `dig` tool to get the CNAME:
- Command: `dig sub.example.com CNAME`
  
Then, you can buy the CNAME and take control of the subdomain.
