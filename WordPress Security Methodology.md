# Understanding WordPress

WordPress is a Content Management System (CMS) that allows users to create and manage websites without needing to build the entire backend from scratch.

WordPress sites share the same core architecture and backend logic, but functionality can differ significantly depending on installed themes, plugins, configurations, and custom code.

If a critical vulnerability is discovered in WordPress core, many WordPress sites may become vulnerable, especially if they are unpatched.

# WordPress Reconnaissance

First we have to know if an application runs on WordPress which we can use a Firefox extension **Wappalyzer** that tells us some information about the (Frameworks, WAFs, Databases, JS library, etc..) that are being used by the application.

Then you can use a tool like **WPscan (WordPress Security Scanner)** which scans the entire WordPress site for outdated versions, exposed components, misconfigurations, and known vulnerabilities.

After creating an account on the WPScan website, you receive an API token that can be used with the scanner.

**WPscan Command *(Run on Linux)*:**
- `wpscan --url https://target.com --disable-tls-checks --api-token <api-token> -e at -e ap -e u --plugins-detection aggressive --force`
(`-e U` --> *Enumerate users*, `--plugins-detection aggressive` -->*Hits more endpoints but is noisier*) 

After the scan finishes, it may returns *interesting entries* related to outdated plugins, vulnerable themes, exposed files, or other potentially vulnerable components.

We can also perform **fuzzing** the target application looking for information disclosure, and information disclosure in WordPress is common because WordPress has so many backup files and a large number of plugins and extensions.

**Example dirsearch command:**
- `dirsearch  -u https://target.com -e conf,config,bak,backup,swp,old,db,sql,asp,aspx,aspx~,asp~,py,py~,rb,rb~,php,php~,bak,bkp,cache,cgi,conf,csv,html,inc,jar,js,json,jsp,jsp~,lock,log,rar,old,sql,sql.gz,sql.zip,sql.tar.gz,sql~,swp,swp~,tar,tar.bz2,tar.gz,txt,wadl,zip,.log,.xml,.js.,.json`

# Common Vulnerability Classes

- Look at **/wp-json/wp/v1/users** or **/wp-json/wp/v2/users** or **/author-sitemap.xml**. *(Mostly patched in modern WordPress installations but you can look for it in older versions)*

- **XMLRPC (`/xmlrpc.php`)** Check if it's enabled: POST with `<methodName>system.listMethods</methodName>` If enabled —> brute force amplification via `system.multicall` which may allow multiple authentication attempts in a single request if rate limiting protections are not properly configured. *(50+ credential pairs in one request, bypasses per-request rate limiting)*

- Also **/wp-content/uploads/** and if **403 forbidden** or **not found 404** then try to bypass it by case-manipulation-->  **/wp-content/UPLOADS/** or **/wp-content/UpLoAds/**. *(e.g. take a look at this reference: https://www.ciel.org/wp-content/uploads/ )*. (*NOTE: this works only on Windows-hosted WordPress (case-insensitive filesystem) — won't work on Linux servers*)

- `admin-ajax.php` is WordPress's backend handler for plugin requests. Some plugins reflect user-supplied parameters without sanitization, making it an XSS target. The payloads below are plugin-specific examples:
	- `site.com/wp-admin/admin-ajax.php?action=tie_get_user_weather&options={'location'%3A'Cairo'%2C'units'%3A'C'%2C'forecast_days'%3A'5<%2Fscript><script>alert(document.domain)<%2Fscript>custom_name'%3A'Cairo'%2C'animated'%3A'true'}`
	- `/wp-content/themes/ambience/thumb.php?src=<body onload=prompt(1)>.png`

- Register via **/wp-login.php?action=register** if there is a login page there is a register that might be hidden, look for it and if found register an account which increases your attack surface. 

- **Login brute force** the `wp-login.php` has no lockout by default *(unless a security plugin is installed)* most Common target: `admin` username *(many installs keep the default).*

- `wp-cron.php` which is publicly accessible by default, and can be abused to trigger resource exhaustion or probe internal behavior.

- **Privilege escalation via role manipulation**: If you can register as a subscriber, look for parameter tampering on profile update requests, changing `role=subscriber` to `role=administrator` is a classic finding in vulnerable plugins.

- **Local File Inclusion (LFI) / Path Traversal**: Vulnerable plugins may improperly handle user-controlled file paths, potentially allowing arbitrary local file reads.  
  
	- *Example payload:*  `http://target.com/wp-content/plugins/mail-masta/inc/campaign/count_of_send.php?pl=/etc/passwd`. If vulnerable, the application may expose sensitive local files such as `/etc/passwd`.

# Disclaimer

This content is intended for educational purposes and authorized security testing only. Do not test systems without proper permission.
