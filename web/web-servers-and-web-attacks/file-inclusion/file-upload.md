# File Upload

{% hint style="info" %}
If your revshell php payload doesn't hit off the bat, start with phpinfo first and work from there
{% endhint %}

We already know

* Null byte poisoning
* Using PHP payloads in an image file (vim) and using both .php and the legit image extension
* Playing around with extensions and encoding
* Case sensitivity (.pHP) or less known extensions (**.phps** or **.php7)**
* Mime Type - doesn't necessarily check for extension name just checks the MIME
* If your file uploaded is non-exectuable try, owerwriting files in locations you'd know (eg. index page, SSH keys etc.)

{% hint style="info" %}
When testing a file upload form, we should always determine what happens when a file is uploaded twice. If the web application indicates that the file already exists, we can use this method to brute force the contents of a web server. Alternatively, if the web application displays an error message, this may provide valuable information such as the programming language or web technologies in use.
{% endhint %}

## Zip Slip

{% embed url="https://medium.com/@instatunnel/zip-slip-the-archive-extraction-vulnerability-everywhere-a37092feb240" %}

## PortSwigger Labs

### Lab 3 (Directory Traversal)

<figure><img src="../../../.gitbook/assets/image (4).png" alt=""><figcaption></figcaption></figure>

In the first two labs, no file extension checking, here only raw content of the files uploaded are being displayed. You can upload the PHP webshell but can't execute commands.

However, there may be a rule that only executes this filter on files uploaded at this specific directory 'avatars'. If we try to escape this then we might be able to execute

`../` was filtered so URL encoding seems to do the trick. Reading the file is as usual

### Lab 4 (Blacklist filter)

{% embed url="https://www.jamesparker.dev/can-i-create-multiple-htaccess-files-for-different-directories/" %}

The fact that different `.htaccess` files can be used for specific directories and their corresponding control is insane.

We can only do that because the response mentions the use of Apache.

Add a `.htaccess` file adding the type 'application/x-httpd-php' basically executable PHP and mentioning our own extension, we bypass the blacklist now.

Why did URL encoding and directory traversal not work or show not found? The response showed it was successful. php5 also only downloaded the uploaded file instead of any execution

<div align="left"><figure><img src="../../../.gitbook/assets/image (5).png" alt="" width="503"><figcaption></figcaption></figure></div>

<div align="left"><figure><img src="../../../.gitbook/assets/image (6).png" alt="" width="504"><figcaption></figcaption></figure></div>

