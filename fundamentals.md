# Hydra Fundamentals

## What is Hydra?

Hydra is an online credential-testing and password-guessing tool that can automate authentication attempts against different network services.

It can be used with services such as:

* SSH
* FTP
* HTTP/HTTPS authentication
* and many other supported protocols

Hydra is mainly useful when we want to test many possible credentials against a live authentication service.

It is important to understand that Hydra is **not breaking the encryption of SSH or HTTPS**.

It is trying credentials against the authentication mechanism and checking the response to determine whether the credentials worked.

---

## Online vs Offline Password Attacks

There is an important difference between online and offline password attacks.

### Online Attack

The target service is running and Hydra sends authentication attempts to it.

Candidate credential → Hydra → Network → Target service → Response

For example:

Username + password → SSH → Accepted / Rejected

Hydra belongs mainly to this category.

### Offline Attack

In an offline attack, we already have a password hash and do not need to repeatedly contact the original authentication service.

Tools such as:

* John the Ripper
* Hashcat

are commonly used for offline password/hash cracking.

The basic difference is:

**Hydra** → tests credentials against a live service

**Hashcat / John** → work against obtained hashes

---

# What Hydra Needs

Before using Hydra, I think about five things:

### 1. Who?

The username or usernames we are testing.

This can be:

* One username
* A username list

### 2. What password?

The password or password candidates.

This can be:

* One password
* A password list

### 3. Where?

The target.

This can be an:

* IP address
* Hostname
* Domain

### 4. How?

How is the target authenticating?

We need to know the service or authentication protocol, for example:

* SSH
* FTP
* HTTP

### 5. How do I know I failed?

Hydra needs a way to determine whether an authentication attempt failed.

This is especially important with web forms because different applications can return different responses for failed logins.

For example:

"Invalid username or password"

or another application-specific response.

So the basic mental model is:

**Who?** → Username

**What?** → Password candidates

**Where?** → Target

**How?** → Authentication service/protocol

**How do I know I failed?** → Failure condition

---

# Credential Attack Techniques

There are several different credential attack techniques that are important to understand.

## 1. Dictionary Attack

A dictionary attack uses a list of candidate passwords.

For example:

* password
* 123456
* qwerty
* admin123
* letmein

Hydra can test these candidates against the target authentication service.

A common example of a password wordlist is `rockyou.txt`.

---

## 2. Brute Force

A brute-force approach generates or tests combinations from a defined character set rather than relying only on a prepared password list.

For example, if a password is known to be exactly four numeric digits:

0000
0001
0002
0003
...
9999

The number of possible combinations is:

10 × 10 × 10 × 10 = 10,000

Brute force becomes expensive as the possible keyspace increases.

---

## 3. Password Spraying

Password spraying means testing one common password, or a small number of passwords, against multiple accounts.

For example:

alice → Welcome2026
bob → Welcome2026
charlie → Welcome2026
david → Welcome2026

This is different from testing many passwords against one account.

The main reason this technique matters is that organizations may have account lockout policies.

---

## 4. Credential Stuffing

Credential stuffing uses previously obtained username/password combinations against another service.

The important difference is that we are not generating random guesses.

We already have credential pairs such as:

user1 → password123
user2 → qwerty123

and test whether users have reused those credentials on another service.

So:

**Dictionary attack** → candidate passwords

**Brute force** → generated combinations

**Password spraying** → few passwords against many accounts

**Credential stuffing** → previously obtained credential pairs

---

# Attack Methodology

Hydra should not be the first thing we run when we see a login page.

The better approach is:

**Reconnaissance → Identify services → Investigate the service → Understand authentication → Check simple possibilities → Choose the appropriate credential technique → Use Hydra if it is suitable → Verify the result**

---

## Step 1 — Reconnaissance

First, we identify what services are exposed by the target.

For example, a scan might reveal:

22/tcp → SSH
80/tcp → HTTP
443/tcp → HTTPS

This gives us information about the attack surface.

The important point is:

> Finding an open port does not mean we immediately attack it.

The open service needs to be investigated first.

---

## Step 2 — Investigate the Service

Suppose we find a web service.

We open the application and discover a login page.

Now we need to understand how the login works.

For a web application, we can inspect the request using a tool such as Burp Suite.

We want to understand:

* Where does the login request go?
* Which parameters contain the username?
* Which parameters contain the password?
* What does a failed login response look like?

For example, we might discover:

POST /login

username=admin&password=test

and the application responds with:

"Invalid username or password"

Now we have information that Hydra can potentially use.

---

## Step 3 — Check Simple Possibilities First

Before using a large password list, check whether there are simpler possibilities.

Some services or applications may have default credentials.

For example:

admin : admin
admin : password

These are only examples. During an authorized assessment, we should use credentials appropriate to the engagement.

If the organization provides test accounts, those should be preferred.

Previously obtained credentials may also be tested where the engagement authorizes credential-stuffing testing.

---

## Step 4 — Choose the Technique

Only after understanding the authentication mechanism should we decide what technique makes sense.

For example:

**Known credential pairs** → Credential stuffing

**Common password across accounts** → Password spraying

**Password candidate list** → Dictionary attack

**Known character set / limited keyspace** → Brute force

Hydra may be useful for some of these situations, but the authentication mechanism determines whether it is actually appropriate.

---

# Websites, Servers and Ports

A common misunderstanding is:

> "Websites don't have ports."

A more accurate way to think about it is:

Server
→ 22 → SSH service
→ 80 → HTTP service
→ 443 → HTTPS service
→ Web application

The web application is accessed through a network service exposed by the server.

So when we perform reconnaissance, tools such as Nmap help us identify the exposed services.

For example:

Nmap
→ 22/tcp → SSH
→ 80/tcp → HTTP
→ 443/tcp → HTTPS

Then we investigate each relevant service separately.

The important idea is:

> **Reconnaissance tells us what is exposed. It does not tell us which attack to perform.**

---

# Hydra Is Not the Answer to Every Login

Seeing a username and password field does not automatically mean:

> Use Hydra.

Modern authentication systems can involve:

* CAPTCHA
* CSRF tokens
* Sessions
* Dynamic requests
* MFA
* API-based authentication
* WAFs
* Rate limiting
* Other authentication mechanisms

These can make simple automated credential testing unsuitable.

Therefore:

> **First understand how authentication works, then choose the appropriate technique.**

---

# My Main Mental Model

The most important thing I learned is to avoid thinking only in terms of commands.

Instead, when I see an authentication service, I should ask:

**Who am I testing?**

↓

**What credentials am I testing?**

↓

**Where am I testing them?**

↓

**What authentication mechanism is being used?**

↓

**How does a failed authentication look?**

↓

**What technique is appropriate?**

The goal is not simply to run Hydra.

The goal is to understand the authentication mechanism and then use the right technique for the authorized assessment.
