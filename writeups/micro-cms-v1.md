# Micro-CMS v1 — Web — 2

## Challenge Info
- **Category:** Web
- **Points:** 2 (2 flags)
- **Files/URL given:** Hacker101 CTF instance URL
- **Description provided:** A simple content management system with pages, creation, and editing functionality. No authentication.

## TL;DR
- **Flag 1:** `^FLAG^4c0f57066b1a1552a4bbf2ac457450992e67720b4e75a05f22510586a1c2cf7b$FLAG$` (hidden admin page)
- **Flag 2:** `^FLAG^743d18cb6432bf139407ddab84514c9c41cc90767b4bec84d1a649b77868eb69$FLAG$` (stored XSS trigger)
- **Vulnerability/trick:** Flag 1 — IDOR via edit endpoint bypasses view restriction on `/page/4`. Flag 2 — Page titles rendered unescaped on homepage allow stored XSS, and triggering XSS detection reveals the flag.

## Recon
- **What I ran first:** Enumerated page IDs at `/page/N`:
  ```
  /page/1  → 200 (public) — "Testing"
  /page/2  → 200 (public) — "Markdown Test"
  /page/3  → 404 (not found)
  /page/4  → 403 (exists, restricted)
  /page/5+ → 404 (not found)
  ```
- **What I found:** `/page/4` returns 403 (exists but blocked). The create page says "Markdown is supported, but scripts are not" — hinting XSS may be relevant.

## Solve Steps
### Flag 1 — Hidden Page via Edit Endpoint
1. The view endpoint `/page/4` is blocked (403), but the edit endpoint `/page/edit/4` has no access control.
2. Request the edit page directly:
   ```
   GET /page/edit/4
   ```
3. The admin page's edit form is returned, containing Flag 1 in the body text.

Why this works: The app checks authorization on the view handler but not on the edit handler — classic IDOR where one endpoint is protected but another isn't.

### Flag 2 — Stored XSS via Page Title
1. Create a new page with a `<script>` tag in the title:
   ```
   POST /page/create
   Content-Type: application/x-www-form-urlencoded

   title=<script>alert(1)</script>&body=test
   ```
2. Visit the homepage `GET /`.
3. The homepage renders page titles without HTML escaping. The XSS detection triggers and appends Flag 2 alongside the script tag in the HTML source.

Why this works: The title field is rendered raw in the page listing (no HTML encoding), and the challenge's XSS detector returns the flag when an XSS payload executes.

## Flag Capture
```
Flag 1: ^FLAG^4c0f57066b1a1552a4bbf2ac457450992e67720b4e75a05f22510586a1c2cf7b$FLAG$
Flag 2: ^FLAG^743d18cb6432bf139407ddab84514c9c41cc90767b4bec84d1a649b77868eb69$FLAG$
```

## Exploit / Script

N/A — manual requests via curl/Burp Suite.
