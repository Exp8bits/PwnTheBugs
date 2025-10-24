- Make a HTML file `test.html`
  ```html
  <html><body>
  <img src="http://YOUR-COLLAB-DOMAIN/track.png" />
  </body></html>
  ```
- Change `test.html` to pdf file `test.pdf`
  ```bash
  # Using wkhtmltopdf
  wkhtmltopdf test.html test.pdf
  # Using pandoc
  pandoc test.html -o test.pdf
  ```
- Upload the PDF to target
  ```bash
  # CURL
  curl -i -X POST -F "file=@test.pdf;type=application/pdf" https://target/upload
  ```
- If the target returns a **URL** after the upload, open that URL in your browser (if it doesn't, check the response body in Burp or Burp history).
- Monitor the HTTP server logs to see if anyone made a GET request for the image.
  - example:
    ```bash
    192.168.1.5 - - [24/Oct/2025 11:23:45] "GET /track.png HTTP/1.1" 200 -
    ```
- Check the headers to determine the request source (`client- vs server-side` processing)
  - Pay attention to:
    - **User-Agent:** if it says `ImageMagick` / `ghostscript` / `Poppler` => most likely a server-side preview
    - **Referer:** may show the file/path that requested the image
    - **Host / X-Forwarded-For:** gives you the original sender IP
