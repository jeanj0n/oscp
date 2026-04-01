# CSRF

We are assuming we have the victim session ID of the corresponding website we are trying to exploit. That's how the browser identifies it as the victim's action and not ours.

Essentially, when a victim clicks a malicious link/payload crafted by us, it sends a payload to website we trying to attack with any particular intended function (eg. change password)

We need victim interaction, maybe some function that clicks on everything we send maybe? you get the point now tho

Assume all steps done to be performed at a victim's browser not ours (eg. changing a cookie is not a inspect element away)

{% embed url="https://brandonrussell.io/OSCP-Notes/CSRF.html" %}

## Brief

For a CSRF attack to be possible

* A relevant action: change users email
* Cookie-based session handling: session cookie
* No unpredictable request parameters

Testing CSRF tokens

1. Remove CSRF token and see if application accepts request
2. Change request from POST to GET
3. See if CSRF token is tied to user session

Testing CSRF tokens and CSRF cookies

1. Check if the CSRF token is tied to CSRF cookie

* Submit invalid CSRF token
* Submit valid CSRF token and cookie from another user

2. Submit valid CSRF token and cookie from another user

TO exploit this,

1. Inject CSRF cookie in user's session (HTTP header injection)
2. Send CSRF attack to victim with known CSRF token

Include PoC from notes
