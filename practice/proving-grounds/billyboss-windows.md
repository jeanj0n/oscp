# Billyboss \[Windows]

## Summary

* Seclists is insane for default passwords for many common applications not just a password dictionary for bruteforcing
* RunasCs.ps1 don't work without RunasCs.exe

## Enumeration

```
sudo nmap -v 192.168.134.61 -sC -sV -p- --open -oN tcpscan.nmap

Nmap scan report for 192.168.134.61
Host is up (0.080s latency).
Not shown: 63258 closed tcp ports (reset), 2264 filtered tcp ports (no-response)
Some closed ports may be reported as filtered due to --defeat-rst-ratelimit
PORT      STATE SERVICE       VERSION
21/tcp    open  ftp           Microsoft ftpd
| ftp-syst: 
|_  SYST: Windows_NT
80/tcp    open  http          Microsoft IIS httpd 10.0
| http-methods: 
|_  Supported Methods: GET HEAD
|_http-server-header: Microsoft-IIS/10.0
|_http-cors: HEAD GET POST PUT DELETE TRACE OPTIONS CONNECT PATCH
|_http-title: BaGet
|_http-favicon: Unknown favicon MD5: 8D9ADDAFA993A4318E476ED8EB0C8061
135/tcp   open  msrpc         Microsoft Windows RPC
139/tcp   open  netbios-ssn   Microsoft Windows netbios-ssn
445/tcp   open  microsoft-ds?
5040/tcp  open  unknown
8081/tcp  open  http          Jetty 9.4.18.v20190429
|_http-server-header: Nexus/3.21.0-05 (OSS)
| http-robots.txt: 2 disallowed entries 
|_/repository/ /service/
|_http-favicon: Unknown favicon MD5: 9A008BECDE9C5F250EDAD4F00E567721
|_http-title: Nexus Repository Manager
| http-methods: 
|_  Supported Methods: GET HEAD
49664/tcp open  msrpc         Microsoft Windows RPC
49665/tcp open  msrpc         Microsoft Windows RPC
49666/tcp open  msrpc         Microsoft Windows RPC
49667/tcp open  msrpc         Microsoft Windows RPC
49668/tcp open  msrpc         Microsoft Windows RPC
49669/tcp open  msrpc         Microsoft Windows RPC
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows

```

#### UDP

```
sudo nmap -sU --top-ports 20 -oN udpscan.nmap -vv 192.168.134.61
Nothing
```

### Files

None

### Ports

#### 21

Need SCP according to the internet

```
ftp 192.168.134.61
Connected to 192.168.134.61.
220 Microsoft FTP Service
Name (192.168.134.61:jtripz): anonymous
534 Policy requires SSL.
ftp: Login failed
```

#### SMB/RPC (139/445)

```
nxc smb 192.168.134.61 -u guest -p '' --shares
SMB         192.168.134.61  445    BILLYBOSS        [*] Windows 10 / Server 2019 Build 18362 x64 (name:BILLYBOSS) (domain:billyboss) (signing:False) (SMBv1:False)
SMB         192.168.134.61  445    BILLYBOSS        [-] billyboss\guest: STATUS_ACCOUNT_DISABLED 
```

#### 80

BaGet - Used to push NuGet packages

Can't fuzz - everything returning 200

<div align="left"><figure><img src="../../.gitbook/assets/image (97).png" alt="" width="563"><figcaption><p>Only interesting part so far</p></figcaption></figure></div>

<div align="left"><figure><img src="../../.gitbook/assets/image (98).png" alt="" width="563"><figcaption><p>Bunch of apis can we do sum w it?</p></figcaption></figure></div>

They all redirect to dead ends.

#### 8081

Find creds for this somehow, the APIs maybe?

<figure><img src="../../.gitbook/assets/image (99).png" alt=""><figcaption></figcaption></figure>

### Exploits

{% embed url="https://www.exploit-db.com/exploits/49385" %}
8081 need creds tho
{% endembed %}

### Loot

#### Creds

`nexus:nexus`

#### Flags

```
83b3c283f968fbfdc0468089adec4abc
ed848a8bb0d9d6ee1b2a9d72d4cb8e44
```

## Initial Foothold

FInd creds for nexus, the other website not useful.

`admin:admin123` don't work and google don't be helping either

<div align="left"><figure><img src="../../.gitbook/assets/image (100).png" alt="" width="563"><figcaption></figcaption></figure></div>

Seclists' `default-passwords.csv` is a large list of default passwords of corresponding services

On looking for Sonatype Nexus, we see another value `nexus:nexus` and it works

`grep -r "Sonatype Nexus"`

<figure><img src="../../.gitbook/assets/image (101).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (102).png" alt=""><figcaption></figcaption></figure>

## PrivEsc

SeImpersonate, use RunasCs.exe first and in that shell use RunasCs.ps1 (UAC bypass) I dunno why but it is what it is

<figure><img src="../../.gitbook/assets/image (103).png" alt=""><figcaption></figcaption></figure>

<div align="left"><figure><img src="../../.gitbook/assets/image (104).png" alt="" width="563"><figcaption></figcaption></figure></div>

## Post Exploitation

<div align="left"><figure><img src="../../.gitbook/assets/image (105).png" alt="" width="409"><figcaption></figcaption></figure></div>
