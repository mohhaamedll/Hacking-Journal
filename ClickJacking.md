# What is Clickjacking:

-Well Clickjacking is when an attacker places an invisible target page underneath a visible decoy page, tricking the victim into clicking something they can't see.

-Well imagine you have your bank account connected to your browser and while browsing on the internet you enter a page saying 'Click Here And You Win 10,000$' the victim clicks not realizing that this is an attacker's decoy page and beneath that button they actually clicked 'Transfer £1000 to the attacker's account' on the real page underneath.

*NOTE: It's a little similar to CSRF but in Clickjacking the victim clicks on buttons that behind it lies another page that the user cannot see, Unlike CSRF where the request is sent automatically without any user interaction, Clickjacking requires the victim to physically click While the attacker just controls what they're unknowingly clicking on.*

--------------------------------------------------------------------------

# How Clickjacking works:
-First the decoy page is just a simple HTML-CSS page that the attackers create, Nothing fancy it just needs to attract the eye and convince the victim to click on the buttons 

-Second the vulnerable web page like the bank account for example,
-  How is the bank transfer page even there? Well it's by using the i-frame mechanism  `<iframe src="https://bankaccount.com/transfer"`.
- The bank i-frame page is manipulated using CSS, The i-frame sits on top of the decoy layer using `z-index:2`, This ensures clicks are intercepted by the i-frame even though it's invisible.
- And is invisible by changing the opacity of the page with  `opacity:0.0001`;
- The i-frame is also precisely positioned using `position: absolute;` with `top: 50px;` and `left:50px;` values so the target button sits exactly beneath the decoy button, pixel-perfect alignment is what makes the victim click the right element. *(these 50px are an example the pixels are changed accordingly to where the buttons are. )* 

And if we want to query something like transferring from the bank page this how it can be done: 
    `<iframe src="https://bankaccount.com/transfer?transaction=1000&transAccount=Hacker.acc"></iframe>`

*NOTE: This example only works if the target action accepts GET parameters, Most real applications use POST requests for sensitive actions, which is why clickjacking is often combined with other techniques.*

--------------------------------------------------------------------------
# Protections:
- `X-Frame-Options: DENY` -> tells the browser to never load this page inside an i-frame.
- `CSP frame-ancestors 'none'` -> the modern version of the same protection.
- Both work by preventing the i-frame from loading in the first place -> no i-frame, no clickjacking. 

--------------------------------------------------------------------------

# Important to say:
- Clickjacking attacks rely on the victim being **already authenticated** to the target website (Logged In).
				 *No login → no sensitive action → no real impact*

