# What CORS:

First CORS stands for Cross Origin Resource Sharing and it's implemented by website's server so that it makes browsers allow some cross origin website to bypass the strict [[SOP]] implemented.** 

-------------------------------------------------------
# How CORS  works:

User sends a request which the browser automatically attaches the requesting page's origin (origin: website.com) via a simple request method like (-`GET,POST,HEAD`-) with no custom headers or a preflight request with custom headers and request method like (PUT,OPTIONS).

-There is a different *RESPONSE* to simple requests & preflight ones but in both the server bring back something like (Access-Control-Allow-Origin : http://example.com  , Access-Control-Allow-Credentials :True | False ),while in preflight requests it also adds (- `Access-Control-Allow-Methods,Access-Control-Allow-Headers`).

-If the website is in the CORS list the browser allow the user who sent the request to read the response

*NOTE: if CORS is badly configured attackers could steal data , CORS is a security mechanism — it's a **relaxation** of SOP done in a controlled way*

-------------------------------------------------------
# Misconfigured CORS: 

- Trusting null origins 

- Suffix / Prefix -> Configuring That if origin starts/ends with myweb.com it is valid so attacker can register a domain like `attackermyweb.com` or `myweb.com.attacker.com` .

- Unescaped Dots -> misconfiguring dots like `/^https:\/\/trusted.com$/` which make it accept a domain like trustedA.com. 

- Subdomain Logics ex: `*.subdomain.com`

- Trusted insecure subdomains + XSS
   
