# CTF Checklist — Quick Reference

## 1. HTML Source Review
- [ ] View page source (`Ctrl+U`) for hidden comments, JS, or flags
- [ ] Check for inline `<style>`, `<script>`, hidden inputs
- [ ] Look at network requests (JS files, API calls, background images)

## 2. Directory & Endpoint Enumeration
- [ ] Fuzz for hidden paths (`/admin`, `/robots.txt`, `/sitemap.xml`, `.git/`, `/api/`)
- [ ] Enumerate numeric IDs: `/page/1`, `/page/2`, ... Look for 403 vs 404 (exists but restricted)
- [ ] Try unprotected edit/create endpoints: `/page/edit/1`, `/page/create`

## 3. IDOR / Access Control
- [ ] Can you access restricted pages via a different endpoint? (view blocked, edit open)
- [ ] Try HTTP method tampering: `PUT`, `PATCH`, `DELETE` instead of `POST`
- [ ] Bypass auth checks by changing `Referer`, `X-Forwarded-For`, `Content-Type`

## 4. SQL Injection
- [ ] Test all input fields (login, search, params)
- [ ] `' OR 1=1-- -` for basic bypass
- [ ] `' UNION SELECT 1,2,3...-- -` — match column count via `ORDER BY N`
- [ ] Time-based blind: `' AND SLEEP(5)-- -`
- [ ] Dump with sqlmap: `sqlmap -u <url> --data="..." --dump --batch`

## 5. XSS
- [ ] Test title/body fields with `<script>alert(1)</script>`
- [ ] Check if output is HTML-escaped or raw rendered
- [ ] Stored XSS → admin/honeypot pages often reveal flags

## 6. Authentication & Sessions
- [ ] Test for weak/default credentials (`admin:admin`, `test:test`)
- [ ] Check cookie flags (HttpOnly, Secure, SameSite)
- [ ] Can you forge or tamper session tokens?

## 7. Data Extraction
- [ ] Database passwords often **are** the flag — dump user tables
- [ ] Error messages may leak info (column names, paths, stack traces)
- [ ] Hidden pages may contain flags in body text

## 8. HTTP Headers & Responses
- [ ] Check response headers for info leakage (`X-Powered-By`, `Server`, custom headers)
- [ ] Look at status codes carefully: `403` = exists, `404` = doesn't, `200` = public

---

**Golden Rule:** If a hint mentions a technique (UNION, methods, credentials), that's your attack vector. Try the simplest thing first.
