# syntax-and-options.md

# Hydra Syntax and Options

## Basic Structure

At a high level, Hydra can be thought of as:

hydra [options] target service

The exact syntax depends on the service being tested.

The important thing is understanding what each part represents rather than memorizing one command.

---

# Credential Options

Hydra uses different switches depending on whether we are providing one credential or a list.

## `-l`

Lowercase -l is used for a **single username**.

Example:

hydra -l admin ...

This tells Hydra that the username is:

admin

---

## `-L`

Uppercase -L is used for a **username list**.

Example:

hydra -L users.txt ...

Hydra can then test usernames from the supplied file.

So:

-l → one username

-L → username list

---

## `-p`

Lowercase -p is used for a **single password**.

Example:

hydra -l admin -p password ...

This supplies one password candidate.

---

## `-P`

Uppercase -P is used for a **password list**.

Example:

hydra -l admin -P passwords.txt ...

Hydra reads password candidates from the file.

So:

-p → one password

-P → password list

The uppercase/lowercase distinction matters.

---

# Target

The target tells Hydra where the authentication service is located.

It can be specified using an IP address or hostname.

For example:

10.10.10.10

or:

target.example

The target is the system where the authentication service is running.

---

# Service / Protocol

Hydra needs to know which service or protocol it should communicate with.

Examples include:

* ssh
* ftp
* http-post-form

Different services have different authentication mechanisms, so Hydra needs to use the appropriate module.

For example:

SSH
→ SSH authentication

FTP
→ FTP authentication

HTTP form
→ Web form authentication

The service is therefore not just another word in the command.

It tells Hydra **how to communicate with the authentication service**.

---

# Example: SSH Structure

A simplified SSH command can look like:

hydra -l admin -P passwords.txt 10.10.10.10 ssh

Breaking it down:

hydra
→ Start Hydra

-l admin
→ Single username

-P passwords.txt
→ Password list

10.10.10.10
→ Target

'ssh'
→ SSH authentication module

The important thing is understanding the role of every part.

---

# Threads: `-t`

The -t option controls the number of parallel tasks Hydra uses.

For example:

-t 4

means Hydra can use four concurrent tasks.

The reason for using multiple threads is that network authentication takes time.

With only one worker, Hydra has to wait for the response before continuing.

With multiple workers, several requests can be in progress at the same time.

This can increase throughput.

However:

> **More threads do not automatically mean better results.**

A target may have:

* Rate limiting
* Firewalls
* WAFs
* Connection limits
* Slow services
* Account lockout policies

Too much concurrency can cause:

* Timeouts
* Connection errors
* Dropped requests
* Rate limiting
* Server overload

So thread count should be chosen based on the target and the rules of the engagement.

---

# Verbose Output

Verbose options can be used when we want more information while Hydra is running.

For example:

-v

can provide more information about the running task.

More verbose output can be useful when troubleshooting a command or trying to understand what Hydra is doing.

The exact amount of output depends on the verbosity level being used.

---

# HTTP Form Syntax

Web login forms are different from simpler services such as SSH because every website can send and respond to authentication requests differently.

For an HTTP POST form, the general structure we learned is:

"http-post-form "PATH:PARAMETERS:FAIL_CONDITION"

For example:

"/login:username=^USER^&password=^PASS^:Invalid login credentials."

This can be broken down into three important parts:

/login
→ PATH

username=^USER^&password=^PASS^
→ PARAMETERS

Invalid login credentials.
→ FAIL CONDITION

---

# ^USER^ and ^PASS^

These are Hydra placeholders.

^USER^

represents the username value Hydra is currently testing.

^PASS^

represents the password value Hydra is currently testing.

For example:

username=^USER^&password=^PASS^

Hydra substitutes the current username and password candidates into these positions when constructing the authentication request.

They are **not** meant to be replaced manually with a specific password list.

The username and password candidates are supplied through options such as:

* -l
* -L
* -p
* -P

---

# How to Find the HTTP Form Parameters

We should not guess the parameters.

A useful approach is to capture a normal login request using Burp Suite.

For example, we may see:

POST /login

username=admin&password=test

From this request we can identify:

/login
→ Login path

username
→ Username parameter

password
→ Password parameter

We can then understand how those values need to be represented in the Hydra HTTP form syntax.

---

# Failure Condition

Hydra needs to know how the application indicates a failed login.

For example, the website might respond with:

Invalid login credentials.

We can use that response as the failure condition.

Conceptually:

Hydra sends credentials
↓
Website responds
↓
Does the response contain the failure condition?
↓
Yes → Authentication failed
No → Investigate as a possible success

This is why understanding the application's response is important before running Hydra.

---

# Why HTTP Forms Need More Investigation

SSH has a defined authentication protocol, so Hydra already knows how to communicate with the SSH service.

Web applications can be implemented in many different ways.

A login could use:

* HTML form
* API request
* JSON
* Session cookies
* CSRF tokens
* Dynamic values
* MFA
* CAPTCHA

Therefore, we cannot assume that every page containing:

Username
Password
Login

can be attacked using the same Hydra syntax.

We first need to understand the actual HTTP request and authentication behavior.

---

# Basic Command Mental Model

When reading a Hydra command, I should be able to identify:

**Hydra**

↓

**Who?**

↓

Username / username list

**What?**

↓

Password / password list

**Where?**

↓

Target

**How?**

↓

Service / authentication module

**How do I know it failed?**

↓

Failure condition

For example:

hydra -l admin -P passwords.txt 10.10.10.10 ssh

means:

Hydra
→ test one username: admin
→ use passwords.txt
→ against 10.10.10.10
→ using SSH authentication

---

# Important

Hydra syntax should not be memorized as a collection of random commands.

The better approach is to understand what information the target requires and then construct the command around it.

Before running a credential test, I should know:

* Target
* Username
* Password source
* Authentication service
* Authentication request
* Failure/success behavior

Then I can decide how Hydra should be configured.
