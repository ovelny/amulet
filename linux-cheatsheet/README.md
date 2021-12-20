# Linux cheatsheet

Set of notes for everything related to Linux & Linux pentesting.

## Navigating the command-line with zsh

With default emacs mode:

* Ctrl+u: delete line
* Ctrl+a: go to beginning of line
* Ctrl+e: go to end of line
* Ctrl+w: delete word backward
* Alt+b: go one word backward
* Alt+f: go one word forward
* Alt+d: delete word forward

## Reading /etc/passwd entries

`goldfish:x:1003:1003:,,,:/home/goldfish:/bin/bash`

* 1. (goldfish) - Username
* 2. (x) - Password. (x character indicates that an encrypted account password is stored in /etc/shadow file and cannot be displayed in the plain text here)
* 3. (1003) - User ID (UID): Each non-root user has his own UID (1-99). UID 0 is reserved for root.
* 4. (1003) - Group ID (GID): Linux group ID
* 5. (,,,) - User ID Info: A field that contains additional info, such as phone number, name, and last name. (,,, in this case means that I did not input any additional info while creating the user)
* 6. (/home/goldfish) - Home directory: A path to user's home directory that contains all the files related to them.
* 7. (/bin/bash) - Shell or a command: Path of a command or shell that is used by the user. Simple users usually have /bin/bash as their shell, while services run on /usr/sbin/nologin.

## Reading /etc/shadow entries

`goldfish:$6$1FiLdnFwTwNWAqYN$WAdBGfhpwSA4y5CHGO0F2eeJpfMJAMWf6MHg7pHGaHKmrkeYdVN7fD.AQ9nptLkN7JYvJyQrfMcfmCHK34S.a/:18483:0:99999:7:::`

* 1. (goldfish) - Username
* 2. ($6$1FiLdnFwT...) - Password : Encrypted password.
	* Basic structure: **\$id\$salt\$hashed**, The $id is the algorithm used On GNU/Linux as follows:
		* \$1$ is MD5
		* \$2a$ is Blowfish
		* \$2y$ is Blowfish
		* \$5$ is SHA-256
		* \$6$ is SHA-512
* 3. (18483) - Last password change: Days since Jan 1, 1970 that password was last changed.
* 4. (0) - Minimum: The minimum number of days required between password changes (Zero means that the password can be changed immidiately).
* 5. (99999) - Maximum: The maximum number of days the password is valid.
* 6. (7) - Warn: The number of days before the user will be warned about changing their password.
