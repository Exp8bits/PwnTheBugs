# Subdomain Takeover Explanation

## Domain Name System (DNS):
- DNS is a server that contains IP addresses and their corresponding domain names.
- DNS is responsible for mapping domains to IP addresses and handling mail routing on the domain.
- The domain name and IP address are essentially two ways of redirecting you to the same destination (server).
- Domain names and IP addresses work together through the use of records.
- The most important DNS records are: A, AAAA, CNAME, PTR, and MX.

### What is an A record?
- **A Record**: Maps a domain or subdomain to an IPv4 address (Subdomain/Domain ⇒ IPv4).
  - Example: `example.com     A     142.250.201.46`
  - `www.example.com` in this case is the subdomain (A record).

### What is an AAAA record?
- **AAAA Record**: Maps a domain or subdomain to an IPv6 address.
  - Same as A record but IPv6 (Subdomain/Domain ⇒ IPv6).
  - Example: `example.com     AAAA     1050:0:0:0:5:600:300c:326b`

### What is a CNAME (Canonical Name) record?
- **CNAME Record**: Maps a domain or subdomain to another domain, often used for cloud services (Subdomain/Domain ⇒ Cloud domain).
  - Example: `example.com     CNAME     example.azure.com`
  - This means `example.com` is an alias for `example.azure.com`. If `example.azure.com` expires, purchasing it can give control over the main site (`example.com`).

### What is an MX (Mail Exchange) record?
- **MX Record**: Maps a domain to an email server (Domain ⇒ Email).
  - Example: `example.com     MX     admin@example.com`
  - Tools: mxtoolbox.com, dig

### What is a PTR record?
- **PTR Record**: Maps an IPv4 address to a domain.
  - A PTR record is essentially the reverse of DNS — it takes an IP address and resolves it back to a domain.
  - Tools: ipython. Example:
    ```python
    import socket
    socket.gethostbyname('TargetDomain')  # Replace TargetDomain with your target domain
    socket.gethostbyaddr('TargetIP')  # Replace TargetIP with your target IP
    ```
  - PTR allows a single IP to map to different domains.

## Known Attacks on DNS:
- **Subdomain Takeover**: 
  - This occurs when an attacker takes over a subdomain or domain name and gains control over it, potentially redirecting it to another location.
- **DNS Spoofing**: 
  - This occurs when an attacker tricks DNS servers into redirecting the victim to a fake website instead of the intended one.
- **DNS Cache Poisoning**:
  - Attackers modify cached DNS data on a server to redirect users to malicious websites.
- **DNS Amplification Attack**:
  - A type of Distributed Denial of Service (DDoS) attack that exploits DNS servers by sending large responses to small requests, overwhelming the target server.

## How Do Subdomain Takeover Vulnerabilities Arise?
- Subdomain takeover vulnerabilities occur when a subdomain is no longer actively managed or is unclaimed. This can happen due to various reasons, such as:
  - **Abandoned Services**: A subdomain may have been used for a service that is no longer active, leaving the subdomain unused.
  - **Misconfigurations**: Errors in managing domains or subdomains can lead to unclaimed subdomains, which are vulnerable to takeover.

## Impact of Subdomain Takeover Vulnerabilities:
- A subdomain takeover exposes an organization’s users and assets to several risks, including:
  - **Misrepresentation**: Attackers could create fraudulent websites under the affected subdomain, deceiving users into interacting with what appears to be a legitimate service or content.
  - **Data Theft**: By controlling the subdomain, attackers can intercept sensitive data transmitted between users and the subdomain, leading to data theft or unauthorized access.
  - **Reputation Damage**: Exploiting a subdomain takeover could harm the organization's reputation, as users may associate the compromised subdomain with security breaches or malicious activities.
