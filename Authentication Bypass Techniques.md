# What Is Authentication:

Authentication is a fundamental security mechanism that strongly supports the CIA triad **(Confidentiality, Integrity, Availability)**.
The bypassing of authentication leads to attacker accessing unauthorized data which breaks Confidentiality, also the attacker bypassing authentication may lead to him changing data which breaks Integrity. 
Authentication is something like login pages which require users to enter their username, password to login to their account, so that the server can authenticate user's identity.
The bypass of authentication occurs when an attacker accesses a user's or admin page they do not own, bypassing the login authentication entirely.
All Applications have an authentication mechanism and in this essay we will learn some bypassing techniques of the login authentication page. 

--------------------------------------------------------------------------
# Authentication Bypass Techniques

- **Host header authentication bypass:**
	*This technique depends on the server trusting the Host header blindly to determine user location or privilege so it allows bypassing of login page, this happens by changing the Host header into `Host:localhost` and then request the `/admin` so the attacker can access the admin panel through the local network range.* 
	*Also if the request line is vulnerable to carrying an absolute URL, some servers may read the host from the request line instead of the Host header, so it could lead to bypassing by entering a full local URL like `GET https://localhost/admin HTTP/1.1` .*

- **Enumeration & Brute Force:** 
    *If the application doesn't have enumeration and brute forcing defenses any attacker could get a wordlist of usernames and passwords and start enumerating and brute forcing the users credentials.*
    Some **Defense mechanism bypass:**
	 ***IP-based brute-force protection:** Could be bypassed easily by manipulating HTTP request headers like adding `X-Forwarded-For`.*
	 *(This only works if the application improperly trusts `X-Forwarded-For` without validating it as coming from a trusted proxy.)*
     
	 ***IP-Block Broken logic:** Some time a attacker's IP gets blocked for some time after multiple incorrect login, but the broken logic that could exist is, that the timer resets every time you login with valid credentials which enable the attacker to brute force by simply adding a macro.*
	 
	 ***Account lock:** Some times an account gets locked during brute forcing if many passwords are used on same username, the bypass is password spraying, trying one common password across many usernames, to avoid triggering the account lockout of any single account.*     
     
	 ***Rate limiting by session/cookie**: Some applications tie rate limiting to a session token rather than IP, This can be bypassed by simply sending requests without a session cookie, or rotating a new one each request. (if rate limiting is only tied to session identifiers)*
	 
  **Username enumeration:**
    *If an attacker doesn't know any usernames to brute force passwords on it, attacker starts to enumerate usernames, by trying a username wordlist on one password and analyze the applications response.
    Responses that can lead to knowing a valid username:
	 **Response timing:** if a username request is valid its response usually takes more time because it checks the password for this correct username, while when username is incorrect it just doesn't need to check the password so the response is faster.*
	 
	 ***Error message differences:** some applications return different error messages like, if "invalid username" message username is wrong but if "invalid password" that means the username is correct.*
	 
	 ***Response length:** even when the error message text looks identical, the HTML response size may differ slightly between a valid and invalid username. Burp Intruder's **Length** column reveals this, valid usernames often produce a response a few bytes longer or shorter than invalid ones.*

- **2FA Bypass:**
    *Some times you have the credentials already but there is a two factor authentication via OTP, so what if:
     **OTP page is misconfigured:**  the server doesn't even validated it so you just put your username, password then request your account home page and bypass the 2FA page.*
     
     ***OTP not tied to session:** when an OTP generated to an attackers own session could be submitted to authenticate a victim's session, that works because the server only check if OTP is valid without checking which session that OTP was issued for.*
     
	 ***OTP Leaked via request**: some times the OTP page has the OTP in its response body or headers leaked when the page loads*
    
	 ***OTP no expire date:** if the OTP doesn't have an expiration date or a short expiration time it can give the attacker time to brute force the 4 digits (assuming no anti-brute-force mechanism is in place).*
     
	 ***OTP response manipulation:** intercepting the failed OTP response and changing the status code from `4xx` to `200`, or changing `"success": false` to `"success": true`, to trick the client side into proceeding. This may only bypass client-side validation and does not affect server-side authentication if properly implemented.*

- **Business Logic Flaws:**
	*Sometimes a application logic is not configured correctly leading to authentication bypass, logic vulnerability differ from an application to another because they are not technical bypass techniques, but flaws specific to how that application's logic was designed.*
	*Example:
		An application has a password reset flow with three steps:
		   Enter your email → server sends a reset token
		   Submit the token → server verifies it
	       Set a new password → server updates the account
	    The broken logic: after verifying the token the server trusts a `username` parameter from the client to decide which account to update. So an attacker can:
			Verify a reset token for their **own** account
			Intercept step 3 and change `username` to the **victim's**
			Server updates the victim's password without questioning it
		No technical exploit,  just the server trusting the client on who to update.*

- **Authentication bypass via information disclosure:**
    *What is an information disclosure? 
	    It's an leaked sensitive information that is found due to the server not configured safely, can be caused by coding bugs or misconfigurations such as insecure settings, exposed debug information, or unintended information leaks like unhidden comments in the code page that can implement how an authentication function logic works, which can help an attacker bypass authentication.*
	
    *A leaked sensitive information can be API keys , internal token, internal headers `(X-Custom-Ip-Authorization: IP)`, authentication mechanism, debug info.*
    
    *How can i obtain leaked info?
     Information disclosure is often actively discovered, not passive, can be found in inspecting  robots.txt file, error messages, debug pages , source code, fuzzing, force errors, inspecting headers/responses intentionally, Just think what might a developer missed here; and know that any sensitive information is useful.*
     
    *Example from A Port swigger lab "Authentication bypass via information disclosure":
     A lab's administration interface has an authentication bypass vulnerability, but it is impractical to exploit without knowledge of a custom HTTP header used by the front-end.
     After knowing this from the lab description we enter by our given account wiener : peter, and start inspecting request in burp suite and manipulate the HTTP headers and methods, and when using the `Trace` method we found that it's available not blocked and that in the returned echo a `X-Custom-Ip-Authorization : [IP]`, that Ip returned was my actively used Ip during the request, knowing that we can figure out that the website uses this header to know if the request is coming from an external/internal Ip and that it blindly trusting it.
     So know we can use burp suite match & replace tool and add the `X-Custom-IP-Authorization: 127.0.0.1` in the replace section, after that click on test and Under Auto-modified request, notice that Burp has added the X-Custom-IP-Authorization header to the modified request then click ok.
     Now all we needed to do is to just try to make a get request to the `GET /admin` and The server treats the request as coming from 127.0.0.1 because it trusts the X-Custom-IP-Authorization header, so the request will enable us access the admin panel, by that we have successfully bypassed authentication using the leaked sensitive info `X-Custom-Ip-Authorization`.*
    
	 *NOTE: The enabled Trace method wasn't mainly a problem until it led us to the trusted header that caused the vulnerability.*



***NOTE:** This essay covers the most common authentication bypass techniques, however authentication vulnerabilities extend further than what is covered here. Topics like JWT attacks and SQL injection authentication bypass are broad enough and can not be covered in this essay, and gonna have their own dedicated essays that will cover them separately.*

--------------------------------------------------------------------------
# Authentication Bypass Impact

Authentication bypass can have severe impact on web applications, by bypassing of authentication attacker can just access admin panel and start deleting user account, Harming application **Availability** and **Integrity** to users, or deleting employees accounts.
An attacker can change data on the application giving misleading **corrupted** information Harming application **Integrity**, and the fact that an attackers bypassed the authentication that's **Confidentiality** breaking.
Even if the authentication bypass happened on a low privilege account "**Still breaking the CIA triad**" that also will still give the victim a bad experience with the application, harming the company's reputation behind that application, this leading to financial losses it self.
So it's important to ensure authentication mechanisms are securely and properly implemented.

--------------------------------------------------------------------------
# Important To Say

These techniques are not a must work, in modern applications defense and protection techniques are hard to bypass, but knowing these technique can help you have clearer and more knowledgeable mindset, with variety of bypass techniques to implement on the target applications.
