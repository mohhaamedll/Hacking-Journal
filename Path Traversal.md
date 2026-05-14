# What is Path Traversal

Path Traversal (also known as Directory Traversal), is one of the most dangerous vulnerabilities that often appears in the **TOP 10** of the **CWE TOP 25** most dangerous software weakness.

Path Traversal vulnerabilities usually happen because the application accepts user-controlled file paths without properly validating or restricting them.

Simply, Path Traversal is a vulnerability that depends on changing the fetched file or directory path from normal like `/var/www/images/1.png` to a file carrying sensitive information like  `../../../etc/passwd`.

--------------------------------------------------------------------------
# How Path Traversal works

As we said earlier we change a file or a directory path to a more sensitive file or directory so first we have to know where an attacker injects the Path Traversal payload.

As normal when searching for a vulnerability an attacker maps the entire application putting notes on:
- weird behaviors
- parameters
- reflected inputs in a URL
- File-related functionality

And also having any proxy *(e.g. Burp Suite)* capturing the requests so you analyze them after mapping.

So in a Path Traversal Vulnerability an attacker looks for a parameter that requests a (file, image, audio, etc..) from a directory, any web application on loading multiple images are fetched from an `/images` directory for the application design or products images, these are where we should try to manipulate and  inject our payload, anything that usually comes from a certain directory may be vulnerable to Path Traversal Attack.


Second, after knowing where the payload is injected an attacker should know what are the sensitive files that should be searched for to acquire sensitive information, there are multiple files that the attacker must know.

*Example:*
-  **/etc/passwd:** *The most common target, containing a list of user accounts and their information.*
-  **/etc/shadow:** *Contains hashed user passwords (often requires root privileges to read).*
-  **/etc/hosts:** *Information about network hosts.*
-  **/etc/ssh/sshd_config:** *SSH server configuration.*
-  **/home/user/.ssh/id_rsa** or **id_rsa.pub:** *User SSH keys for unauthorized remote access.*
-  **/var/log/apache2/access.log** or **/var/log/httpd/access_log:** *Web server access logs, useful for further exploitation.*
-  **/root/.bash_history:** *Command history of the root user.*

*NOTE: The file you inject in your payload is decided according to the attackers need and the application he is attacking.*

*NOTE: These are not the only files but some of the common ones, you can always search for the exact file you want depending the type if application you are attacking.*


Third, Mostly when you see a Path Traversal payload you see a `../../../` before the exact file path that becomes when a request look something like this:

- *GET /?image=1.jpg --> This image path is actually `/var/www/images/1.jpg`*

So if changed the **1.jpg** into `/etc/passwd` the payload will run something like this `/var/www/images/etc/passwd` and the application just returns no such file, and that true because there is no such file called `/passwd` or even a directory called `/etc` inside the `/images` directory, so the `../` is needed because it tells the application to go up a directory so a full payload example to demonstrate.

*Example:*
- *GET /?images=../../../etc/passwd* --> which the first ../ will get us out of the `/images` directory, the second `../` gets us out out the `/www` directory and the third gets us out of the `/var` directory which ends up from this `/var/www/images/` into the Root Directory `/` which is the main directory and from it you can start to enter any directory you want so the application resolves the payload to `/etc/passwd` and fetch the passwd file from the `/etc` directory.

*NOTE: It's recommended to do more than 3 `../` because the path from the current directory and the main root directory can vary to having 3,4,5+ directories in between so always so don't give up after just one payload the application still can be vulnerable to path traversal, you just haven't gotten the correct payload yet.* 

--------------------------------------------------------------------------
# Bypassing Path Traversals Defenses

After Finding the parameter to inject and understanding how the payload works and knowing what sensitive files to search for you would think that if you tried the path traversal attack on any application it would work, and well that's wrong most applications now have strong WAFs, firewalls that protect them and they mostly also apply defense mechanisms against the Path Traversal Attack.

But that doesn't mean it's bulletproof. Not every defense mechanism is ideally configured neither correctly.

So here are some misconfigured defense bypassing techniques:
- **Absolute Path:** *Simply an application could be taking the input as the absolute path so just inject `/etc/passwd` without any `../`.*

- **Non-Recursive Stripping:** *An application could be stripping certain values like `../` but not recursively so injecting something like `..././` where `../` will be stripped and the `. & ./` get reattached forming another `../` so an attacker bypasses stripping if not recursive.* 

- **Encoding & Decoding:** *An application WAF, Firewall could just be blocking certain characters that are suspicious, but it still could be misconfigured so that it doesn't decode the input so instead of just injecting `../` you inject `..%2F` which can bypass the filter as being a safe input but if blocked you can still try **double encoding** which if the WAF or Firewall decodes the input only once then a payload double encoded like this `..%252F` will be decoded to `..%2F` and pass the filter as safe input while after reaching the backend  it gets decoded to `../` and runs as malicious. This works because there are two decoding layers: the WAF decodes the input once (seeing `%2F`, a safe-looking value) and passes it through, then the backend decodes it a second time resolving it to `../` and interpreting it as a path traversal sequence. 
  Example payload: `%252e%252e%252fetc/passwd`.*
  *NOTE: Mostly the blocked in path traversal are the `../` not the single slash `/`.*

- **Validation of Path Start:** *If an application has a weak validation function for the path start like for example if must have `/var/www/images` in the input so you just inject a `/var/www/images/../../../etc/passwd` bypassing the validation function.*

- **Null Byte at File Extension:** *If an application has a weak validation function for the file extension you just inject a payload with a null byte validating the extension that doesn't violate the payload like `../../../etc/passwd%00.jpg` the `%00.jpg` just validate the extension without changing the payload (mostly works on old systems, backend that is written in C/C++ or older PHP (pre-5.3.4)).*

*NOTE: These are some bypassing techniques and if they all doesn't work separately it doesn't mean that the application is secure always try mixing between them.*

*NOTE: These techniques work on misconfigured defenses which is something common but an ideally configured strong defense will require advanced techniques or it just could be path traversal not the way to go with this application.*

--------------------------------------------------------------------------
# Impact 

The Path Traversal attack can lead to severe damage to an application, as we said earlier it's mostly in the **TOP 10** in the **CWE TOP 25** and it's rated mostly above **Six** and considered Medium or High.

Information like usernames, UIDs, home directories, and shells in the `/etc/passwd` and hashed password in `/etc/shadow` can lead to account hijacking which severely damages the *Confidentiality* in the **CIA** triad.

So if a Path Traversal Attack is discovered in an application, it should be patched quickly so that no attacker can abuse the discovered vulnerability, leading to damage to an application's reputation or costing the application financial losses.

--------------------------------------------------------------------------
# Mitigations Against Path Traversal

As we said the Path Traversal attack could lead to a lot of damage so here are some mitigation techniques to protect against Path Traversal:
- **Non User Input:** *Not making an application take input from a user is always the best practice but if that's not possible you can look at other techniques.* 
- **Whitelist:** *An application can implement a whitelist of the only allowed directories and blocking any others so something like `/etc/passwd` gets blocked entirely.*
- **Strong Validating:** *Making a strong validation function to validate every user input to prevent any malicious input.*
- **canonicalize the path:** *Using some functions like e.g. `getCanonicalPath()` in Java, `realpath()` in PHP/Python, canonicalizing the path before validation so traversal sequences like `../../../etc/passwd` resolve into their real path, then verify the resolved path stays inside the allowed directory.
