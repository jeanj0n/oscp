# AD + Windows

{% embed url="https://github.com/OoStellarnightoO/OSCP_Notes/blob/main/12-ActiveDirectory.md" %}

{% embed url="https://github.com/omarexala/OSCP-Notes/tree/master/active-directory/internal-enumeration-and-lateral-movement" %}
I don't know much about these ACLs I'm pretty sure bloodhound takes of this but PowerView there in case
{% endembed %}

## MS01

Do this again if you find a new user

* Privs & Groups - Check User Description for a clue (b.martin was SQL Admin remember?)
* 3 D's (Downloads, Documents & Desktop)
* `dir /s /b` the current Users' directory
* PowerUp to find easy wins (Services, Default creds)
* `cmdkey /list`
* PS history
* Web Server?
* Uncommon Installed apps (SSH, PuTTY etc. - find private key, plaintext password in registry)
* Check protocols - (SMB, RPC etc.)
* Get user list & password reuse
* Kerb + ASREP (enumusers first obv)
* Bloodhound (First Degree Control etc.)
* WinPEAS - _**registry**_ & others
* Scheduled tasks
* Watch for registry stuff from BEN

#### Admin (Refer Cool Stuff I Found)

* mimi (Get EVERYTHING, cache, lsa, sam, logonpasswords - yk the drill)
* PS history
* .kdbx loot, browse entire Admin folder

Now SPRAY SPRAY SPRAY

cleartext and NTLM, local and domain accounts, all protocols - you will get a hit

## MS02

* Ligolo tunnel
* Port forward if internal ports (Don't ever forget)
* Repeat MS01
* Scan ports with and without ligolo

## DC01

* GPO is affecting the whole domain, if it's a vector in MS01 you'd get domain compromise instantly - also the default domain policy only resides in dc
