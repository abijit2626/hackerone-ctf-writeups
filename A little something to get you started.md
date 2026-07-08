# A Little Something to Get You Started — Hacker101 CTF Write-Up

## Overview

A basic web challenge serving as an introduction to Hacker101 CTF. The flag is hidden directly in the page source.

---

## Flag — HTML Comment

**Hint:** Look at the source.

The challenge page displays a simple message: *"Welcome to level 0. Enjoy your stay."*

### Steps

1. Open the page in a browser.
2. View the page source (right-click → View Page Source, or `Ctrl+U`).
3. The flag is hidden in an HTML comment:
```html
<!-- ^FLAG^...$FLAG$ -->
```

### Tools Used

- Browser developer tools
- curl

| Flag | Technique | Difficulty |
|------|-----------|------------|
| Flag | View Source | Trivial |
