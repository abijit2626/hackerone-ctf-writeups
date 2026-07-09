# A Little Something to Get You Started — Web — 1

## Challenge Info
- **Category:** Web
- **Points:** 1
- **Files/URL given:** Hacker101 CTF instance URL
- **Description provided:** A simple web page displaying "Welcome to level 0. Enjoy your stay."

## TL;DR
- **Flag:** `^FLAG^5e1f77d850c7c774c7ed1e46cfbe894fbb74ec021fbe56f8e56f5735c51fc0a9$FLAG$`
- **Vulnerability/trick:** Flag hidden in an HTML comment in the page source.

## Recon
- **What I ran first:** Opened the page in a browser — just a paragraph element with welcome text and a background image.
- **What I found:** The page source (Ctrl+U) revealed an HTML comment containing the flag.

## Solve Steps
1. Visit the challenge URL.
2. View page source (right-click → View Page Source or `Ctrl+U`).
3. Read the HTML comment:
```html
<!-- ^FLAG^5e1f77d850c7c774c7ed1e46cfbe894fbb74ec021fbe56f8e56f5735c51fc0a9$FLAG$ -->
```

No exploitation needed — the flag is directly visible in the source.

## Flag Capture
```
^FLAG^5e1f77d850c7c774c7ed1e46cfbe894fbb74ec021fbe56f8e56f5735c51fc0a9$FLAG$
```
