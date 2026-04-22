# LFI

* System is accessing a local resource
* User input is being downloaded

## LFI Hitlist

{% hint style="info" %}
Use cURL if content displayed on webpage is messy (no spaces or required tabs for SSH keys etc.
{% endhint %}

* etc/passwd -> user SSH keys \[clock algo from nmap] in `home/[user]/.ssh/id_rsa`\
  (SSH key algo used can be determined from nmap scan need not always be RSA eg. ECDSA)
*   **Apache webroot** -> \[htaccess and htpasswd?] any config files retrieved will need this first<br>

    ```
    Check the 'Web Servers & Web Attacks' page for all locations for config files
    /etc/apache2/apache2.conf
    ```
* Webapp creds
* Service creds (could be of another port eg. redis or whatever)
* `.bash_history/.zsh_history/.sh_history` of users who have most odds of running the app (find location of stuff)
* `/proc/self/cmdline`  \
  `/proc/self/environ`  \
  `/proc/self/status`  \
  `/proc/version`
* `C:\inetpub\wwwroot\web.config`
* `/etc/apache2/sites-enabled/000-default.conf`

{% embed url="https://sirensecurity.io/blog/file-inclusion-reference/" %}
SAM & SYSTEM are there here
{% endembed %}

{% embed url="https://github.com/RoqueNight/LFI---RCE-Cheat-Sheet" %}

{% embed url="https://sushant747.gitbooks.io/total-oscp-guide/content/local_file_inclusion.html" %}
Sensitive file list
{% endembed %}
