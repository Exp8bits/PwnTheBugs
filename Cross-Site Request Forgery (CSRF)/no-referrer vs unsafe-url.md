## What are the differences between **no-referrer** and **unsafe-url**?
```html
<meta name="referrer" content="no-referrer">
```
⇒ This setting completely removes the referrer information, meaning the referrer header will not be sent with requests.
⇒ Use case: Enhanced privacy such as sensitive information (e.g. page structure or query parameters) should not be shared.
```html
<meta name="referrer" content="unsafe-url">
```
⇒ This setting sends the full URL of the referring page in the referrer header for both secure (HTTPS) and insecure (HTTP) requests.
⇒ Warning: This could leak sensitive URL components (e.g. query parameters) to insecure destination.
⇒ Use case: Typically discouraged unless specifically required.
