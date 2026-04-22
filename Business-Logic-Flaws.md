# What are Business Logic Flaws:

Business logic flaws (BLF), referred to in OWASP as a part of **Insecure Design**, are not technical vulnerabilities. Instead, they are logical vulnerabilities that arise from flawed assumptions in how the application is designed to function.

They occur when developers assume users will behave in a certain way, but an attacker intentionally breaks those assumptions. This often involves sending unexpected or manipulated requests that the application fails to properly handle.

**Example:**
	In a shopping web application, the attackers found that when adding a product to the cart the price is sent with the request body. When the request is intercepted and modified, the price changes in the cart, allowing the attacker to purchase the item at an unintended price.

--------------------------------------------------------------------------
# Types of BLF:

Business Logic can be many things, but it always leads to the breaking of developers' trust. Here are the main categories:  

- Excessive trust in client-side controls
  _(e.g. price manipulation - leading to buying items with unintended prices.)_

- Failing to handle unconventional input
  _(e.g. negative quantities - can lead to buying items at a lower price, like adding a jacket with $1000 and adding 10 negative items with $100, ($1000-10*100) = 0$ . )_

- Flawed workflow assumptions
  _(e.g. skipping payment steps - leading to completing an order without going through payment because you can skip the checkout page and go directly to the order on its way page.)_

- Domain-specific flaws
  _(e.g. abusing discount logic - leading to buying an item with unintended prices by putting a promo code more than once or two promo codes after another, or maybe the page has its own gift card leading to buying a $10 gift card for $7 after a promo code, then receiving $10 cashback, netting $3 profit per cycle, repeatable infinitely.)_

- Providing an encryption oracle:
	  (where the application inadvertently gives attackers the ability to forge encrypted values using the server's own encryption functionality)

--------------------------------------------------------------------------
# Methodology BLF:

Finding Business Logic flaws is different from other vulnerabilities it doesn't require writing code or using payloads. It depends on the attacker thinking like a developer.
To think like a developer you need to go through everything a developer went through, so map the whole application, understand how it works, know the application's boundaries, workflow, parameters.
Think about how the developer wants the workflow to be and go against it, skip sequences, test boundaries, do unusual cases.
Intercept requests, start manipulating values, removing values, and watch how the application reacts to your changes.
Send old requests to the repeater and try them after logging out, after the session expires, try everything that comes to your mind and keep hunting.

*NOTE: Always ask yourself what did the developer trust here? Every parameter you intercept, every workflow step you skip, every boundary you test, the answer to this question is your attack surface*

--------------------------------------------------------------------------

# Impact & Remediation:

The impact of Business Logic flaws can vary from one logic bug to another, a bug can lead to unauthorized access to an admin panel, which damages the company's confidentiality, integrity, and availability if an attacker deletes any user or manipulates data.
Any logic bug can hurt a company depending on its severity, but usually it leads to financial loss whether from direct exploitation or the cost of patching and incident response. Also, a logic bug can lead to privilege escalation, and data exposure.

Preventing Business Logic Flaws starts from the designing phase, Developers should enforce workflow state server-side, never trust client-supplied input, and validate all values, including price, quantity, and role, on the server regardless of what the client sends. Unlike technical vulnerabilities, logic flaws cannot be patched with a library update; they require redesigning the flawed assumption entirely.

--------------------------------------------------------------------------

*Business Logic flaws are a reminder that security is not only about writing secure code, it's about thinking securely from the start.
As long as developers make assumptions about how users behave, attackers will find a way to violate these assumptions.
The best defense is building an application that enforces its own rules, not ones that rely on users following them.*
