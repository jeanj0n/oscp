# API

{% embed url="https://github.com/danielmiessler/SecLists/tree/master/Discovery/Web-Content/api" %}

{% hint style="info" %}
Monitored \[HTB Linux]\
XposedAPI & Nickel \[PG]
{% endhint %}

* Request method (GET, POST) - use common sense when to use what lol
* Are we missing data for the full request (eg. 500 Internal Server Error)
* Authorization to access the endpoint (eg. apikey, basic auth)
* The screenshots below show even a slash can make a big difference in response, do not crash out over sum like that

## Fuzzing

Actions and objects (verbs and nouns respecitvely) are important to scope the full functionality of an API endpoint available if the documentation is horrible.

API paths are often followed by a version number, resulting in a pattern such as:

```
/api_name/v1
/api_name/v2
```

You might need to be authenticated, a value such as an `apikey` may be required for successful fuzzing

Change request methods to see if the same endpoint works

This was just fuzzing, interacting with it is another ball game

## Interacting

**Basic Authorization:**

Example from stripe:

```
curl https://api.stripe.com/v1/charges -u [apikey]:
```

curl uses the -u flag to pass basic auth credentials (adding a colon after your API key will prevent it from asking you for a password).

**Custom Header**

```
curl -H "X-API-KEY: value" https://api.mydomain.com/v1/users
```

```
feroxbuster -u https://nagios.monitored.htb/nagiosxi/api -m GET,POST -k
feroxbuster -u https://nagios.monitored.htb/nagiosxi/api/v1 -k --query apikey=value -w /opt/SecLists/Discovery/Web-Content/api/objects.txt

For GET requests, do as is standard stuff
ffuf -w [wordlist] -u [URL] -X POST -d 'id=FUZZ' -H 'header: value' -fs xxx  
(Use raw request) : ffuf -request request.txt -request-proto http[s] -w wordlist.txt -r [FUZZ should be present in the request]
```

<figure><img src="../../.gitbook/assets/image (10).png" alt=""><figcaption></figcaption></figure>

<div align="left"><figure><img src="../../.gitbook/assets/image (11).png" alt=""><figcaption><p>Even a '/' makes a big difference</p></figcaption></figure></div>
