# Clickjacking Vulnerability

##  What is Clickjacking?
Clickjacking is a type of attack where an attacker tricks a user into clicking on something different from what the user perceives. This is usually done by loading a vulnerable webpage inside an invisible or opaque frame and overlaying it with fake elements.

##  Detecting Clickjacking
- Install **Click-jacking** extension by **Daoud Youssef**.
- This extension highlights vulnerable web pages with a **red border** if:
  - They are vulnerable to Clickjacking.
  - They **lack the X-Frame-Options header**.

If a web page is vulnerable, it will show a red box. You can then create a **Proof of Concept (PoC)** using the following HTML code:

##  Proof of Concept (PoC)
```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Clickjacking PoC</title>
    <style>
        body {
            margin: 0;
            padding: 0;
            background-color: #f4f4f4;
            display: flex;
            justify-content: center;
            align-items: center;
            height: 100vh;
        }
        .container {
            position: relative;
            width: 800px;
            height: 600px;
            border: 2px solid red; /* Red box to highlight the frame */
            overflow: hidden;
        }
        iframe {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            opacity: 0.3; /* Makes the target page slightly transparent */
            pointer-events: none; /* Disables direct interaction with the frame */
        }
    </style>
</head>
<body>
    <div class="container">
        <iframe src="http://target-site.com"></iframe>
    </div>
</body>
</html>
```

##  When is it a vulnerability?
- If the **iframe** successfully loads data from the target site, then the site is **vulnerable**.
- If the site displays **sensitive data** (such as account details, personal data, admin panels, etc.) and lacks **X-Frame-Options**, it should be reported immediately.

>  **Note:** If the site allows framing but doesn’t display sensitive data, the issue might only be rated as **Informative**.
