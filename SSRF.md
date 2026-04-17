
# What is SSRF:

SSRF stand for Server Side Request Forgery, it's an attack technique that happens on the server side to fetch information.
this attack thoroughly depend on the trust between services, so for example there is mostly no defenses between a back end and a database communication because of the trust relation between them.

---------------------------------------------------------------------
# Types of SSRF:

In-Band vulnerability (Reflected) is whenever you get a response for your request like fetching info,
in a lab environment, exploiting SSRF against an internal admin panel returned the admin page HTML directly in the HTTP response, confirming in-band retrieval.
Out-Of-Band vulnerability Blind SSRF produces no visible response to the attacker. You can detect it by making the server send a request to an external server you control, like a Burp Collaborator endpoint, and watching for incoming DNS lookups or HTTP requests.
In a blind SSRF lab, sending a Burp Collaborator URL as the stockApi parameter produced no visible response, but the Collaborator panel received a DNS lookup confirming the server made an outbound request to the attacker-controlled endpoint.

---------------------------------------------------------------------
# SSRF Targets:

Localhost/internal services
Cloud metadata endpoint (`169.254.169.254`)
Internal APIs and databases
Alternative protocols (`file://`, `dict://`, `gopher://`)

---------------------------------------------------------------------
# How SSRF Works:

SSRF first works on the pages that fetch data from a server like a quantity checker that shows how many products left, so intercept this request and change the fetch order from `ex: stockapi=http://prodect/stock/check?prodId=1&shopId=1` to `http://localhost/admin`, this work because of the TRUST relationship between back end & localhost.

---------------------------------------------------------------------
# SSRF Defenses:

SSRF defense can be a white list or a black list, where black list can be bypassed easily because it only blocks some character and there is tons of different payloads that can pass it, while white list is harder because the fetch request has to be designed in a certain way like `"External stock check host must be stock.weliketoshop.net`, this mean that the request must look like this
	        `http://stock.weliketoshop.net/pagename/pagename'`
Which can be played around like by : 
- you can embed credentials in a URL before the hostname, using the `@` character. For example:
    `https://expected-host:fakepassword@evil-host`
- You can use the `#` character to indicate a URL fragment. For example:
    `https://evil-host#expected-host`
- You can leverage the DNS naming hierarchy to place required input into a fully-qualified DNS name that you control. For example:
    `https://expected-host.evil-host`
- You can URL-encode characters to confuse the URL-parsing code. This is particularly useful if the code that implements the filter handles URL-encoded characters differently than the code that performs the back-end HTTP request. You can also try double encoding characters; some servers recursively URL-decode the input they receive, which can lead to further discrepancies.

- *NOTE: These techniques can be combined — for example, double URL encoding `#` as `%2523` and placing the whitelisted host after `@` tricks the validator into seeing the allowed host, while the HTTP client decodes the fragment marker and routes the request to the internal target instead:*  
		`http://localhost%2523@stock.weliketoshop.net/admin`"
---------------------------------------------------------------------
*NOTE: Successful SSRF can lead to sensitive data exfiltration, internal network pivoting, cloud credential theft via metadata endpoints, and in some cases full remote code execution.*
