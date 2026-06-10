# What Is A JWT

Modern web applications need to verify user identity in every request which is done using session tokens, but in every request there is a database check for this session token each time which kills scalability. JWTs solve this problem by making a token that doesn't need a continuous database  lookup.  

JWT is short for JSON Web Token, JWT is a token that works as an authentication mechanism for users, it replaces normal server-side sessions that **need** a database check every time while JWTs don't which increase performance and gets rid of unnecessary operations.
#### HEADER, PAYLOAD, SIGNATURE

A JWT is a container that consists of three parts `HEADER.PAYLOAD.SIGNATURE`, that is Base64URL encoded sent in the `Authorization: Bearer <token>` Header.

The **Header** carries info about the token itself like:
- `alg: algorithm`
- `typ: type`
- `kid: key-id`

While the **Payload** carries a set of data known as claims *(Registered Claims, Private Claims, Public Claims)*.
**Registered Claims:**
- `iss: Issuer`, *The entity which created and signed the token.*
- `sub: Subject`, *The identity of the token, usually the user id.*
- `exp: Expiration Time`, *The time left before the token expires.*
- `iat: Issued At`, *The timestamp of when the token was created.*
- `aud: Audience`, *The intended app or API that the token is authorized for.*

**Private Claims:** *they are mostly user related claims*
- `role:`, *The role of the user.*
- `id:` , *The Id of the user*.

**Public Claims**: *These are custom claims created by developers or organizations.*

While the **Signature** which is responsible for verifying integrity of the JWT contains a cryptographic signature generated from the Header and Payload.

**The Signing Process:** 
1. Base64Url-encode Header.
2. Base64Url-encode Payload.
3. Concatenation of Base64Url-encoded Header and payload with a dot in between.
4. Signing that string using the algorithm in the header.

***Example** of a JWT:*
- `eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6IkpvaG4gRG9lIiwiYWRtaW4iOnRydWUsImlhdCI6MTUxNjIzOTAyMn0.KMUFsIDTnFmyG3nMiGM6H9FNFUROf3wh7SmqJp-QV30`

***Important Note:** JWTs are encoded, not encrypted. Anyone who obtains a JWT can decode and read its Header and Payload. The Signature only provides integrity and authenticity; it does not provide confidentiality.*

--------------------------------------------------------------------------
# How Does JWT Work

To know how JWT works, we should first understand what a cookie is and what is the sequence of the authentication process.

#### Normal Cookies
First a user login to their accounts and the server returns a session cookie that authenticates the user so that there is no need to log in every process. 

So How Does the server check on this cookie, well the server takes the session cookie runs a database check on it to know what account this session is from and allow the passing of requests and process if the token is valid.
#### JWTs
So as we said earlier that JWTs do not need any database check, so how is that? To answer this question first we should know that in the server creation of a JWT it doesn't save it.

Also as we said the JWT is signed, and this signing happens with a **Key** that can be symmetric *(HS256)* or asymmetric *(RS256)*.

So in the **checking** process the server takes the full JWT and **recalculate** the `HEADER.PAYLOAD` signature with the **key** it has and try to match the **JWT Signature** that is sent with the request with the **Server Calculated signature**, if they match then the JWT token is valid and if they don't match then the JWT has been modified.

*NOTE: The JWT can't be modified because the attacker doesn't have the signing key so any modification results in the unmatched request signature with the calculated signature, Most Importantly security guarantee depends entirely on the signing key staying secret.*

--------------------------------------------------------------------------
# JWT Attacks

#### Mindset

JWT attacks depend on the misconfigurations from developer at the development of the JWT authentication mechanism, not a technical attack using a tool that can bypass any JWT authentication. 

After understanding this you should focus on thinking and asking your self what a developer may have forgotten or wrongly implemented and try to exploit these.

#### Tools
There are variety of ways to interact, modify, resign the JSON Web Tokens, you can use *(**JWT.io tool**, **Burp Suite Extension Tools** (JSON Web Tokens, JWT Editor), **Personal Generated Python tools** (Advanced))*.

- **JWT.io:** It is a JWT online tool that you can edit your JWT `Header.Payload.Signature` in it's debugger section, and it's easy and simple to work with.

- **JSON Web Tokens & JWT Editor:** These are burp extension tools that you can install from **BApp Store** JSON Web Token shows you the JWTs sections separated and decoded and allows you to edit on them, while JWT Editor tool allows Generate signing keys and analyze tokens.

- **Personal Generated Python tools:** This is when you write the code that takes the JWT separate it and decode it by your self instead of using automated tools, it's a little advanced but you can add you personal signature cracking to the code and make it even better than ready automated tools.

#### Attacks
The following attacks are ranked from least to most common.

- **Unverified Signature:** *Happen when there is a mistake in the verification code where a developer forgot or mistyped verify function `verify()` to any other like decode function `decode()`, as we said it's possible but not common, but in this situation an attacker base64Url-decode the JSON Web Token and modify in the payload section to change the `role: user` to `role: admin` or the `sub: username` to `sub: admin` without changing the signature and sending the request, which will allow admin privileges because there is not verifying on the signature.*
	- ***Reference:** You can solve this scenario in this [Port Swigger lab](https://portswigger.net/web-security/jwt/lab-jwt-authentication-bypass-via-unverified-signature).*

- **Flawed Signature Verification:** *This can happen if there is a verification function but its flawed to algorithm changing to none  `alg: none` this happen if the the server blindly trusts the request header and doesn't validate the algorithm on the server side (The JWT spec originally allowed `alg: none` for tokens already verified by a trusted party, which is still honored by some libraries by default), so as there is no algorithm used you delete the sent signature, and now you modify the request payload from `role: user` to `role: admin` or `sub: username` to `sub: admin`.*
	- ***Reference:** You can solve this scenario in this [Port Swigger lab](https://portswigger.net/web-security/jwt/lab-jwt-authentication-bypass-via-flawed-signature-verification)*.

- **Authentication bypass via weak signing key:** *This can happen if the application used key is a weak key which can lead to a weak signature that an attacker can break and extract the key from it by brute forcing it using tools like **Hashcat** or **jwt_tool**, so if we have weak signature we take the full JWT and run this hashcat command `hashcat -a 0 -m 16500 <JWT> /wordlists/jwt.txt` which will bring back the secret key used in signing the JWT, now if we have the key we can modify the request and resign it using that key all this can be done in the (Burp suite **JSON Web Token** tool)*.
	- ***Reference:** You can solve this scenario in this [Port Swigger lab](https://portswigger.net/web-security/jwt/lab-jwt-authentication-bypass-via-weak-signing-key)*.

- **JWT authentication bypass via JWK header injection:** *This can happen if the the server blindly trusts the JWK (JSON Web Key) header and doesn't validate the key on the server side, a vulnerable server reads the `jwk` parameter directly from the token header and uses it to verify the signature, meaning it trusts a key the attacker provided, so now we can generate a key using the **JWT Editor** by clicking (generate new RSA key) or (generate new symmetric key) depending on the algorithm used (`alg: RS256` or `alg: HS256`), lets say that the application use an `alg: RS256` so we click generate new RSA key and take it as a Jwk format, then go to the JWT header in the (JSON Web Token tool) and add `"jwk": {json web key in jwk format}`and then modify the request payload and resign it with the new generated RSA Key, and send the request.*
	- ***Reference:** You can solve this scenario in this [Port Swigger lab](https://portswigger.net/web-security/jwt/lab-jwt-authentication-bypass-via-jwk-header-injection)*.

- **JWT authentication bypass via JKU header injection:** *This can happen if the the server blindly trusts the JKU (JSON Key URL) header and doesn't validate the key on the server side, a vulnerable server reads the `jku` parameter directly from the token header and uses it to verify the signature, meaning it trusts a key the attacker provided, this is the same as the JWk header injection but instead of passing the key in the header section in a jwk format, you put a URL of your own website like this `"jku": "https://myownjku.com"` in this URL there is the key in this format  `{ "Keys": [ {jwk format of the key} ] }`.*
	- ***Reference:** You can solve this scenario in this [Port Swigger lab](https://portswigger.net/web-security/jwt/lab-jwt-authentication-bypass-via-jku-header-injection).*

- **JWT authentication bypass via kid header path traversal:** *This happens when the server doesn't validate the `"kid": "key id"` header and blindly trusts it which can lead to injecting in the **kid** a **path traversal payload** `../../../../../../../../../dev/null`, which point to a null value, then we generate a null key, we can do that using **JWT editor** by clicking on generate symmetric key and click generate and in the `"k"` field we add `AA==` which is base64 encoded of a single zero byte (null), the `"k"` should look like this `"k" : "AA=="`, so after generating a zero byte key and a changing the **kid** `"kid": "../../../../../../../../../dev/null"` to point to a null value we modify the payload to what we want and resign the request with our new generated key.*
	- ***Reference:** You can solve this scenario in this [Port Swigger lab](https://portswigger.net/web-security/jwt/lab-jwt-authentication-bypass-via-kid-header-path-traversal)*.

- **JWT authentication bypass via algorithm confusion:** *This vulnerability occurs when an application expects JWTs to use an asymmetric algorithm such as `RS256` but blindly trusts the `alg` value supplied by the user.  So if the **algorithm** used is **RS256** which is asymmetric so it signs the data using a private key and verifies it using a public key, and the public key is often publicly accessible can be in an endpoint like `jwks.json`, so what happens when we change the **algorithm** to **HS256** which is a symmetric algorithm so the key used for verifying will be assumed by the server to be the signing one as well, so i will take the public key which will be in a **jwk** format so we will go to **JWT Editor** and generate a new RSA key first pass our public key in the key section and then click on the **PEM format** take it and encode it to base64 then we create a symmetric key by clicking on the generate symmetric key, then click generate and in the ("k") i will add the base64 encoded PEM format of the public key and save the key, after that we just modify the header algorithm as we said and modify the payload to what we want and resign it with our new generated symmetric key and send the request.
	- ***Reference:** You can solve this scenario in this [Port Swigger lab](https://portswigger.net/web-security/jwt/algorithm-confusion/lab-jwt-authentication-bypass-via-algorithm-confusion)*.

***NOTE:** If you can't obtain the public key you can use the `portswigger/sig2n` tool it works by taking two different JWTs signed with the same RSA private key and uses them to mathematically derive the server's RSA public key, you can run the tool using docker with this command `docker run --rm -it portswigger/sig2n <JWT1> <JWT2>`.*

--------------------------------------------------------------------------
# Impact

A misconfigured JWT mechanism can lead to severe issues, a successful JWT attack gives the attacker one of these outcomes.
- **Authentication Bypass:** Accessing unauthorized endpoints or functionality.
- **Privilege Escalation:** Elevation of attackers privileges from a normal user into admin by changing the `sub` or `role` claim.
- **Account Takeover:** Forging a token for any user by controlling the payload.

In Bug Bounty Programs, JWT vulnerabilities are rated **Medium**, **High** and **Critical** severity due to the direct impact on **authentication integrity**.
