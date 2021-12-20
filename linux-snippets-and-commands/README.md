# Linux snippets and commands.md

Snippets and commands for Linux pentesting.

## Enumeration

```bash
whoami
uname -a
groups
id
ifconfig
ip addr
env
netstat -tulpen
cat /etc/passwd
cat /etc/shadow
cat /etc/*-release
ps aux | grep root
hostname
dpkg -l
find / -writable -type f 2>/dev/null
lsmod
/sbin/modinfo <driver_name>
```

## Exploitation

TBA.

## Privilege escalation

```bash
# find SUID binaries, compare with GTFObins for privesc techniques
find / -perm -u=s -type f 2>/dev/null

# grant you instant root access if no password is required
sudo -i

# show privileges that could be exploitable
sudo -l

# list cron jobs, look out for wildcards or globbing patterns
cat /etc/crontab
ls -lah /etc/cron*
```

### Easy method for shell stabilization

```bash
# in attack box
nc -lvnp 4444

# in your reverse shell
python -c 'import pty; pty.spawn("/bin/bash")'
export SHELL=bash
export TERM=xterm-256color
Ctrl-Z

# back in attack box
stty raw -echo; fg
reset
```

## Useful commands

```bash
# trim all whitespace from input
echo '<paste-here>' | tr -d " \n\t\r"
```
