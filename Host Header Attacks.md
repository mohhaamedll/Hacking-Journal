# What Is Host Header:

To understand the Host header you have to know why there is Host headers in the first place? And that's because the fact that one IP can host multiple websites (domains) so the Host header is there to specify which web page you want.
Example:
    `8.8.8.8 --HOST--> google.com , supp.google.com , email.google.com` 

Host headers are sent via requests and it looks something like this `Host: example.com` , when the server processes the request, it checks for the Host header to determine which web page is requested and start sending the response.  

*NOTE: HTTP Host headers are very important and must be well configured or it could lead to multiple vulnerabilities.* 

--------------------------------------------------------------------------
# Host Header attacks:

As we said before the misconfigurations' or lack of protection for the Host header can lead to many vulnerabilities like: 

- Password Reset Poisoning:
	*Vulnerability occur in password reset functionality, the main idea works by the attacker sending a change password request with the victims username but manipulates the Host header to his own server so that the link gets sent back to the attacker so the attacker access the victims password reset page*
  ***Root Cause**: Server uses Host header to build reset URL without validation*

- Authentication bypass:  
	*Main idea works by manipulating the Host header to the local host and fetching the admin page so the server thinks that the request is from the local network and allows the attacker to bypass authentication, and the request looks like this:
			GET /admin HTTP/1.1
			Host: localhost
  **Root Cause**: Server trusts Host header to determine if request is internal*

- Web cache poisoning via ambiguous requests:  
	 *Main idea is that The cache and the back-end server disagree on what the "cache key" is. So Attacker sends ambiguous request with extra Host header, Cache keys on first Host header (legitimate), Back-end processes second Host header (attacker's), Poisoned response gets cached under legitimate key, and all victims get attacker's response.*
	 *Ambiguous Request look something like this:
         GET /  HTTP/1.1
         Host: legitimate-site.com
         Host: attacker.com*
  ***Root Cause**: Cache and back-end read duplicate headers differently*

- Routing-based SSRF: 
	 *Vulnerability exists due to infrastructure that uses Host headers to route requests to different back end servers By manipulating the Host header, the attacker can make the system route the request to an internal service that should never be publicly accessible.*
  ***Root Cause**: Load balancer routes based on Host without restricting internal network range*

These attacks can be tested using tools like **Burp Suite** for intercepting and modifying requests, and `curl` for crafting custom HTTP requests.
		`e.g. curl -H "Host: attacker.com" https://target.com` 

*NOTE: If Host header can't be manipulated or injected you can still manipulate the host via proxy headers or alternative headers like `X-Forwarded-Host(Proxy) , X-Host , X-Forwarded-Server`  these headers can still be vulnerable if the back-end trust them without validation, even if the main Host header is not.*

--------------------------------------------------------------------------
# Mitigation of Host Headers:

- Validate a Host header against a whitelist of allowed domains.
- Don't trust `X-Forwarded-Host` unless it comes from a trusted proxy.
- Sanitize untrusted headers.
- Configure cache to include full Host header in cache key.
- Never use Host header to build URLs without sanitization. 
