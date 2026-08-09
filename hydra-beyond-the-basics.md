# Hydra Beyond the Basics

## Password Spraying

Password spraying is different from a normal dictionary attack.

In a dictionary attack, we usually focus on one account and try many passwords.

Example:

admin → password1
admin → password2
admin → password3
admin → password4

In password spraying, we reverse the approach.

We take one password and try it against multiple accounts.

Example:

alice → Welcome2026
bob → Welcome2026
charlie → Welcome2026
david → Welcome2026

The idea is to avoid repeatedly attacking the same account.

This is particularly relevant when an organization has account lockout policies.

---

## Credential Stuffing

Credential stuffing is another technique that is often confused with brute force.

Here, we already have username/password combinations obtained from another source.

For example:

alice : password123
bob : Summer2025!
charlie : qwerty123

We then test whether those exact credentials work on another service.

The assumption being tested is:

> Users sometimes reuse passwords across different services.

Credential stuffing is therefore not about guessing passwords.

It is about testing previously obtained credential pairs.

---

## Default Credentials

Default credentials are credentials supplied by a vendor, manufacturer, application, or administrator when a system is initially configured.

Examples could include:

admin : admin

or another vendor-specific default combination.

The important point is that default credentials are not necessarily "weak passwords" in the traditional sense.

They may simply never have been changed after installation.

During an authorized assessment, checking for known default credentials can therefore be useful during the early stages of testing.

---

## Authentication Enumeration

Authentication enumeration means determining information about accounts or authentication behavior based on how a system responds.

For example, imagine a login system behaves differently for:

Invalid username

and:

Correct username + incorrect password

If the application gives noticeably different responses, an attacker may be able to determine which usernames exist.

This creates an information disclosure problem.

A secure application should generally avoid revealing unnecessary information about whether a specific account exists.

---

## Rate Limiting

Rate limiting restricts how frequently a client can perform an action.

For authentication, this might mean limiting login attempts from:

* An IP address
* An account
* A device
* A session
* Another identifying factor

For example:

Too many attempts
↓
Slow down requests
↓
Temporary restriction

Rate limiting is one of the main defenses against automated online credential attacks.

It also means that simply increasing Hydra's thread count is not necessarily useful.

---

## Account Lockout Policies

An account lockout policy can temporarily or permanently restrict an account after a defined number of failed authentication attempts.

For example:

5 failed attempts
↓
Account locked for 15 minutes

This can make traditional brute-force attacks less practical.

However, lockout policies also need to be designed carefully.

An overly aggressive lockout policy can itself become a denial-of-service problem if someone intentionally triggers lockouts for many accounts.

This is one reason password spraying is an important concept to understand.

---

## MFA

MFA stands for **Multi-Factor Authentication**.

Instead of relying only on:

Something you know → password

MFA can require another factor, such as:

Something you have → security key or phone

Something you are → biometric factor

For example:

Username + Password
↓
One-time verification code
↓
Access

A valid password alone may therefore not be enough to authenticate.

This is one of the reasons Hydra does not magically bypass modern authentication systems.

---

## CAPTCHA

CAPTCHA systems are designed to distinguish automated requests from human interaction.

For example, a login system might require the user to complete a CAPTCHA after several failed attempts.

This can interfere with automated credential testing.

The important lesson is:

> The presence of a username and password field does not mean the entire authentication process can be automated with Hydra.

---

## WAF Detection

A WAF, or **Web Application Firewall**, sits between clients and a web application and can inspect incoming traffic.

It may detect patterns associated with:

* Automated requests
* Suspicious request rates
* Repeated login attempts
* Known attack patterns
* Malicious payloads

If a WAF detects suspicious activity, it may:

* Block requests
* Rate-limit the client
* Return a challenge
* Log the activity
* Temporarily block an IP address

This is another reason why blindly increasing Hydra's speed can make a test less reliable.

---

## Session-Based Authentication

Modern web applications often use sessions.

A simplified authentication flow might look like:

User submits username and password
↓
Server verifies credentials
↓
Server creates a session
↓
Server gives the browser a session identifier
↓
Browser sends that identifier with future requests

The application then knows that the user has already authenticated.

This is different from a simple request where every request independently contains a username and password.

Understanding session behavior becomes important when analyzing modern web applications.

---

## CSRF Tokens

CSRF stands for **Cross-Site Request Forgery**.

A CSRF token is often a unique value included in a request to help the application verify that the request came from an expected context.

A simplified form might contain:

username

password

csrf_token

The token may change between requests.

This creates a problem for simplistic automated requests because the tester cannot necessarily send the same request repeatedly.

The authentication request may require a fresh token each time.

---

## Dynamic Login Forms

Not every login form is a simple:

username + password → response

Modern applications may use:

* JavaScript
* AJAX/fetch requests
* JSON
* Dynamic parameters
* CSRF tokens
* Session cookies
* Multiple authentication steps

For example:

Browser loads login page
↓
Server generates dynamic value
↓
JavaScript processes the form
↓
Browser sends API request
↓
Server validates credentials
↓
Session is created

If we only look at the visible webpage, we may misunderstand how authentication actually works.

This is why inspecting the actual HTTP traffic is important.

---

## Basic Authentication

HTTP Basic Authentication is a different authentication mechanism from a typical HTML login form.

A browser may display a username/password prompt directly rather than providing a normal webpage form.

The credentials are sent as part of the HTTP authentication process.

This is one of the authentication mechanisms that tools such as Hydra can interact with because the protocol is standardized.

The important distinction is:

**HTML login form**

→ Application-specific request

**HTTP Basic Authentication**

→ Standard HTTP authentication mechanism

---

## NTLM

NTLM is an authentication protocol historically associated with Microsoft environments.

It uses a challenge-response mechanism rather than simply sending a password directly to the server.

A simplified idea is:

Server sends challenge
↓
Client uses credential-derived information
↓
Client sends response
↓
Server verifies response

NTLM is commonly encountered in Windows-based environments and internal networks.

It is important because authentication in enterprise environments is often more complicated than a simple username/password form.

---

## Hash Cracking vs Online Cracking

These are fundamentally different approaches.

### Online Credential Testing

The authentication service is running.

Hydra sends credentials to the service and observes the response.

Example:

Hydra
↓
SSH server
↓
Authentication response

The target's defenses can therefore interfere with the process.

These include:

* Rate limiting
* Account lockouts
* MFA
* WAFs
* Network controls

### Offline Hash Cracking

We already have a password hash.

Instead of contacting the original service repeatedly, we attempt to determine the original password from the hash.

Tools such as:

* Hashcat
* John the Ripper

are commonly used for this purpose.

The important distinction is:

**Online cracking**

→ Interacts with the authentication service

**Offline cracking**

→ Works against obtained password hashes

---

## Why Hydra Won't Magically Work Against Every Login Page

This is probably the most important concept beyond the basic Hydra syntax.

A login page does not automatically equal a Hydra-compatible target.

Hydra needs to understand the authentication mechanism it is communicating with.

A simple web form might look like:

username=USER
password=PASS

But a modern application might require:

username
password
CSRF token
session cookie
dynamic parameter
MFA code

The request may also be sent as JSON rather than normal form data.

For example:

Browser
↓
GET /login
↓
Server provides session + CSRF token
↓
Browser submits login request
↓
Server validates token
↓
Server validates credentials
↓
MFA challenge
↓
Authenticated session

A basic Hydra HTTP form module cannot automatically understand every possible application workflow.

---

## The Right Question to Ask

Instead of asking:

> "Can I use Hydra against this login page?"

I should first ask:

> "How does this authentication mechanism actually work?"

Then investigate:

* What request is sent?
* Which parameters are required?
* Is a session required?
* Is there a CSRF token?
* Does the request change dynamically?
* What indicates failure?
* What indicates success?
* Is MFA involved?
* Is there rate limiting?
* Is there a WAF?
* Is the authentication actually HTTP form-based?

Only after understanding these things should I decide whether Hydra is appropriate.

---

## Hydra's Place in a Pentest

Hydra is one tool in the authentication-testing toolbox.

It is useful when the target's authentication mechanism is compatible with the modules Hydra provides.

It is not a universal password-cracking solution.

A simplified decision process is:

Reconnaissance
↓
Identify authentication service
↓
Understand authentication mechanism
↓
Check for simple/default credentials where authorized
↓
Determine whether online credential testing is appropriate
↓
Choose dictionary attack, password spraying, credential stuffing, or another technique
↓
Select the appropriate tool
↓
Test carefully
↓
Verify the result

The tool comes **after understanding the target**, not before.

---

## Final Mental Model

The biggest lesson from Hydra is not memorizing commands.

It is understanding authentication.

When I encounter a login system, I should think:

**What am I authenticating to?**

↓

**How does the authentication mechanism work?**

↓

**What does a valid request look like?**

↓

**What does a failed request look like?**

↓

**What defenses are present?**

↓

**What testing technique is appropriate?**

↓

**Is Hydra actually the right tool?**

That mindset is much more valuable than simply knowing how to type a Hydra command.
