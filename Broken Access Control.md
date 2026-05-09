## What Is Access Control

Before jumping into Broken Access Control techniques we should first understand what access control is.

Access control is the mechanism responsible for putting limitations on what a user can access, do, or configure within an application.

- *Example: The admin panel can't be accessed by a normal user, the payout confirmation page can't be accessed before a checkout page, the account of a certain user can't be accessed without the user's cookie or credentials.*

The misconfiguration of access control mechanisms leads to vulnerabilities known as **Broken Access Control (BAC) vulnerabilities.**

---------------------------------------------------------------------
## Types of BAC

- **Vertical Privilege Escalation:** *When an attacker can access resources or functions of a higher privilege user --> a normal user accessing the admin panel.*

- **Horizontal Privilege Escalation:** *When an attacker can access resources or data belonging to another user --> a normal user accessing another user's resources.*

- **Context-Based Broken Access Control:**  *When an attacker can bypass the sequence of functions  --> accessing payout confirmation page before a checkout page.*

- **Insecure Direct Object References (IDOR):** *When an attacker can access resources or data belonging to another user by changing user ID in the request --> after authentication changing user ID parameter from attacker ID to victim ID.* 
*NOTE: IDOR is not only limited to the changing of user ID, it can involve any direct object reference (File ID, Order ID, Document ID, etc..).*

---------------------------------------------------------------------
## How To Find BAC Vulnerabilities

While searching for Broken Access Control you should always think of what a developer might have missed or misconfigured, in most applications there are BAC vulnerabilities that are not yet discovered.

**First** you should map the entire application and take your time doing so, you can't find BAC in an application you don't know how it works and functions, while mapping the application, use a proxy to intercept and capture requests to analyze every request that could have something like a *(File ID, Order ID, Document ID, Sequence ID, Any reference parameter or ID)*.
Also after mapping and analyzing requests take a look at the source code if you have access to it, doing so can help you understand how access control functions work exactly, and if the developer didn't practice secure coding you might discover multiple vulnerabilities faster.  

**Second** Start manipulating the normal request by changing IDs, trying to access unauthorized pages *(e.g. admin panel)*, if you have access to an admin account (White-Box), try resending all admin panel functionality requests while being unauthenticated, try again while being authenticated with a normal user, if you are testing a shopping application analyze how its checkout functionality works, try requesting functions not in the right sequence.

**Always** send unusual requests and values and analyze the error messages, any information disclosure about the defense mechanism can give you ideas of how to bypass them, and always think of how the developer expected users to interact with the application, then try interacting with it in unexpected ways.

**Third** if you found a direct object reference, try changing the ID if it prevents access, don't give up try sending the request but in different ways.

- **Example:** */API/V5/ID/5 and you are manipulating the ID, but just changing it from 5 to 6 don't do anything, then start trying different ways the same ID can still be vulnerable.*
-    Try:
		- *Trailing Slash: /API/V5/ID/10/*.
    
		- *Double Slash: /API/V5/ID//10/*
    
		- *Version Downgrade: /API/V1/ID/10*
    
		- *Sub Path / End Point Variant: /API/V5/ID/10/Details*
    
		- *Multi ID / Filter Parameter Abuse:  /API/V5/ID/10,9 or /API/V5/ID/10.9 or  /API/V5/ID/010*
    
		- *Type Confusion: `/API/V5/ID/*10*` or /API/V5/ID/10abc*
    
	  *NOTE: Try combining multiple techniques together* 
    
*NOTE: Most of these techniques where found effective in finding BAC vulnerabilities while solving educational port swigger Broken Access Control labs, CTF challenges and  Bug hunting.*

*NOTE: These methodologies can help you find BAC vulnerabilities but it doesn't mean they will work on all applications, it's just for helping you understand and have some guidance of what to do.* 

---------------------------------------------------------------------
## Impact

Broken Access Control seems simple but in fact it is very common, making it the **Number ONE** vulnerability in **OWASP TOP 10 2021**, and has been in the top 10 for years as access control mechanisms are commonly misconfigured and Broken Access Control can appear in many different forms and scenarios.

Broken Access Control severely harms the **CIA** triad (Confidentiality, Integrity, Availability), harming Confidentiality if an attacker could access victim data, harming both Confidentiality and Integrity if an attacker could access an admin panel and manipulate functions within the application, harming Confidentiality, Integrity and Availability if an attacker could access a user account and delete the user's account.

Broken Access Control severity can vary from low to critical, though BAC vulnerabilities are often considered high severity, which should be patched quickly so that no attacker can abuse the discovered vulnerability, leading to damaging an application's reputation or costing the application financial losses.

---------------------------------------------------------------------
## Mitigations

As we saw in previous sections Broken Access Control can severely damage an application, so here are some main mitigations that can be implemented to prevent BAC and IDOR vulnerabilities.
- **Server-Side Validation**:  *Always validate everything on the server side.*
- **Deny-by-Default Access Control**: *If no explicit access control rule exists, the application denies the request by default.* 
- **Least Privilege Enforcement:** *Always apply the minimum required privileges to all users.*
- **Implementing Role-Based Access Control:** *RBAC is a mechanism that identifies each user's role and assigns access rights based on that role.*
- **Proper Session Management:** *Applying proper session management is crucial, so it ensures that authenticated users have timely and appropriate access to resources, thereby reducing the risk of unauthorized access to sensitive information.*
- **Secure Coding Practice:** *Writing secure code is crucial so that the developer should never trust a user's input and must always implement validation and sanitization mechanisms to ensure user input can't cause any harm, and avoid using insecure functions or libraries.*
