# Micro-CMS v1 — Hacker101 CTF Write-Up

## Overview

Micro-CMS v1 is a simple content management system with pages, creation, and editing functionality. The challenge has no authentication — anyone can create and edit pages.

---

## Flag 1 — Hidden Page via ID Enumeration

The CMS uses numeric page IDs (`/page/1`, `/page/2`, etc.). Brute-forcing IDs reveals a hidden page at `/page/4` which returns **403 Forbidden** (it exists but is restricted).

### Steps

1. Enumerate page IDs:
```
/page/1  → 200 (public)
/page/2  → 200 (public)
/page/3  → 404 (not found)
/page/4  → 403 (exists, restricted)
/page/5+ → 404 (not found)
```

2. The edit endpoint (`/page/edit/4`) does **not** have the same access control. Access it directly:
```
GET /page/edit/4
```

3. The admin page's content is fully visible in the edit form, including the flag in the body text.

**Alternative:** POST to `/page/edit/4` to overwrite the admin page's content.

---

## Flag 2 — XSS via Page Title

**Hint:** *"Markdown is supported, but scripts are not"*

While the page says scripts are not allowed, the homepage renders page titles without proper HTML escaping. Creating a page with a `<script>` tag in the title triggers the XSS detection.

### Steps

1. Create a new page with a `<script>` tag in the title:
```
POST /page/create
title=<script>alert(1)</script>&body=test
```

2. Visit the homepage (`/`). The title is rendered unescaped in the page listing, and the flag appears alongside the injected script.

The flag is visible directly in the HTML source of the homepage.

---

## Tools Used

- Browser developer tools
- Burp Suite / curl
- Page ID enumeration

## Flag Summary

| Flag | Technique | Difficulty |
|------|-----------|------------|
| Flag 1 | Hidden page via edit endpoint | Easy |
| Flag 2 | Stored XSS via page title | Easy-Medium |
