# A Little Something to Get You Started

## Challenge Information
- Category: Web
- Difficulty: Easy
- Platform: Hacker101
- Points: 1

## Challenge Description
A simple web page displaying: *"Welcome to level 0. Enjoy your stay."*

## Initial Analysis
- Nothing is displayed on the page itself, so the flag must be hidden in the page source or assets.
- The simplest hiding place for a flag in a basic challenge is an HTML comment.

## Enumeration

### Step 1 — View the page source

Open the page and view the source (right-click → View Page Source, or `Ctrl+U`).

```html
<!doctype html>
<html>
    <head>
        <style>
            body {
                background-image: url("background.png");
            }
        </style>
    </head>
    <body>
        <p>Welcome to level 0.  Enjoy your stay.</p>
    </body>
</html>
```

The flag is not visible in the rendered output but appears in the raw HTML.

## Exploitation

### Step 1 — Find the flag

View the page source. The flag is hidden in an HTML comment:

```html
<!-- ^FLAG^...$FLAG$ -->
```

## Flag

```
^FLAG^...$FLAG$
```

**Technique:** View Source
**Difficulty:** Trivial
