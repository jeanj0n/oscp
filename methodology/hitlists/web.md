# Web

TAKE SCREENSHOTS OF EVERYTHING EVEN IF IT LOOKS USELESS

IF YOU SPOT A VECTOR AND IT DON'T WORK, LOOK OUTSIDE YOUR NOTES

THE HINTS WILL BE ON SCREEN (RESPONSE, ERRORS)

* Scope entire application first - fuzz all directories
* Now include extensions based on wappalyzer output
* If an application is running, version number and check for public exploit
* For creds, try default {admin,user,\[app\_name],offsec:admin,password,\[app\_name],offsec}\
  If still nothing, Seclists or hydra
* SQLi auth bypass, code exec if you do clock it
* List all endpoints taking user input
* By now, you'll see a play - don't buy the bait just yet
* Page source via curl since dev tools is a pain
* If you see a domain used, watch for a subdomain either in source or fuzz
* Request headers and tokens (eg. JWT)
* SSRF for URL input values and SSTI for them templates
* Do NOT stress about APIs, look at the response codes and request methods and data that can be sent and functions performed
* Flask (main.py for app), Werkzeug (WSGI) these things give more clarity for the scope
