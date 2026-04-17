
# What is CSRF:

First CSRF stand for Cross Site Request Forgery, it's a cross site attack that occurs on the client side
and works on changing data on victim's account like changing (Email , password , Username ) ,CSRF vulnerability
only works if the change page doesn't require old password or email it will work if the attacker knew them but if
not CSRF isn't the way to go but if the victim's page do not ask for re-authentication before applying change it
might work if CSRF protection is misconfigured.

--------------------------------------------------------------------------
# How CSRF works:

It works by attackers having a Web page with a submit form in it,
that when a victim enter the web page the form is submit opening the victim's account on the vulnerable website.

Now you ask your self how does the the submitted form enter the victim's account on the vulnerable website,
that works if the victim's has logged in into there account before so the browser send the session cookie with the
submit form on the victims machine that end up opening the vulnerable website and victim's
account and applying the change the attacker wanted.

It seem a little complicated so here is a simple graph to simplify things a little:

`[victim] --Enters(1)--> Attacker Web page --Form submits(2)--> Go to Vulnerable website with browser attaching the victims cookie with the request
                           --changeing victims info(3)--> CSRF Attack Complete`

--------------------------------------------------------------------------

# CSRF Protections: 

[1] CSRF tokens -> Is a Token that is Generated for each session ideally per request and is attached with each request 
so if the attackers website doesn't sent the CSRF token of the victim with the submit form the request will not be permitted,
The CSRF-Token can't be forged because of SOP preventing the attacker from reading responses to extract victims token.   

[2] Same-Site Policy ->it tells the browser _don't send this cookie on cross-site requests at all_.
So the forged request reaches the server but arrives with no session cookie,making it unauthenticated and harmless.

[3] SOP (Same Origin Policy) ->Browsers Preventing attackers from reading responses to extract sensitive values like CSRF tokens,
which protects against CSRF indirectly. *(Talked about more in SOP Essay)*.

