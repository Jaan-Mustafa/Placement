# URL Shortener — How Redirection Works

How a short link like `https://tinyurl.com/33anywxt` redirects to the original long URL.

## The Short Code

```
https://tinyurl.com/33anywxt
                    └──┬──┘
                   short code (the unique key)
```

`33anywxt` is just a **unique key**. In the database there's a row:

```
short_code  →  long_url
─────────────────────────────────────────────
33anywxt    →  https://www.example.com/some/very/long/path?x=1...
```

The whole job: take `33anywxt`, look up the original URL, send the browser there.

## The Redirect Flow (Step by Step)

```
1. You click  https://tinyurl.com/33anywxt
        │
        ▼
2. Browser → DNS → TinyURL's server (request: GET /33anywxt)
        │
        ▼
3. Server extracts the code "33anywxt" from the path
        │
        ▼
4. Server looks up the code in DB/cache:
       "33anywxt" → "https://www.example.com/some/very/long/path"
        │
        ▼
5. Server responds with an HTTP REDIRECT:
       Status: 301 (or 302)
       Location: https://www.example.com/some/very/long/path
        │
        ▼
6. Browser sees the redirect → automatically makes a NEW request
   to the long URL
        │
        ▼
7. You land on the original page. ✓
```

The key step is **#5 — the HTTP redirect response.** The server doesn't show content; it tells the browser "go here instead", and the browser follows automatically.

## What the HTTP Response Looks Like

```http
GET /33anywxt HTTP/1.1
Host: tinyurl.com

      ↓ server responds ↓

HTTP/1.1 301 Moved Permanently
Location: https://www.example.com/some/very/long/path?x=1
```

That `Location` header is the entire trick — the instruction the browser obeys.

## 301 vs 302 — Which Redirect?

| | 301 Permanent | 302 Temporary |
|---|---|---|
| Meaning | "This short URL *always* maps here" | "Map may change, ask me each time" |
| Browser behavior | **Caches** it — next click may skip TinyURL's server entirely | Does **not** cache — always hits TinyURL's server |
| Pro | Faster, less load on server | You can track every click (analytics) |
| Con | Can't count clicks (browser bypasses you) | More load on server |

> TinyURL-type services often use **302** so they can **count clicks / do analytics** on every visit. With 301 the browser caches the jump and they never see repeat clicks.

## How the Short Code Gets Created (the other half)

```
1. You submit: https://www.example.com/some/very/long/path
2. Server generates a unique short code: "33anywxt"
      (via: base-62 encoding of an auto-increment ID,
       or a hash of the URL, or a counter like Snowflake)
3. Server stores:  33anywxt → https://www.example.com/...
4. Returns to you: https://tinyurl.com/33anywxt
```

**Base-62** is common: codes use `[a-z A-Z 0-9]` = 62 characters. With 7 chars → 62⁷ ≈ **3.5 trillion** unique URLs. That's why codes are short yet plentiful.

## Why It's Fast at Scale

The lookup (`33anywxt` → long URL) happens **billions of times**, so:
- **Cache** (Redis) holds hot mappings → lookup in <1ms instead of hitting the DB.
- The mapping is read-heavy and rarely changes → perfect for caching.
- DB is often a **key-value store** (`code` is the key) → fast point lookups, easy to shard by code.

## One-Line Summary
**The short code is a key; the server looks it up in a database to find the original long URL, then sends the browser an HTTP 301/302 redirect with a `Location` header pointing to that long URL — and the browser automatically follows it.**

---

## Practical Note: "I don't see any request in the Network tab"

A real observation: clicking a TinyURL sometimes shows **no request to tinyurl.com** in DevTools → Network. The request *does* happen — you just don't see it. Reasons:

### Reason 1: Network tab clears on navigation (most common)
Clicking the link **navigates away** to the long URL. By default DevTools **wipes the Network log on every navigation**, so the tinyurl request was recorded for a split second, then cleared.
**Fix:** DevTools → Network → check ☑ **Preserve log** (Firefox: gear → "Persist Logs"). Re-try and you'll see:
```
39vbj87m        301 (or 302)   document   ← the redirect
some-long-url   200            document   ← the final page
```

### Reason 2: 301 redirects get cached by the browser
If TinyURL returned **301 Permanent** the first time, the browser **cached the mapping**. On later visits it jumps **straight to the long URL without asking tinyurl.com again** → no request appears at all.
**Prove it:** open **Incognito** (fresh cache) → request shows up. Cached ones show **"(from disk cache)"** in the Size column.

### Reason 3: DevTools opened *after* navigating
Network only records while open. Open DevTools **first**, then paste the link.

### Reason 4: Address-bar paste vs clicking a link
Address-bar navigation still makes the request, but with the log clearing on navigation (Reason 1) it vanishes instantly unless Preserve log is on.

### Definitive proof via curl
```
$ curl -sI "https://tinyurl.com/39vbj87m" | grep -iE "HTTP/|location"

HTTP/2 301
location: https://www.reddit.com/r/pune/comments/1ue5s83/does_anyone_know_siya_goyal/
```
The `301` + `location` header prove the redirect happens. The browser just hid it due to **301 caching + the network log clearing on navigation** — not because there was no request.

---

## 301 vs 302 — Deeper Comparison

Both are redirects (both send a `Location` header the browser follows). The difference is **permanence**, which changes caching:

| | **301 Moved Permanently** | **302 Found (Temporary)** |
|---|---|---|
| Meaning | "Resource moved **forever** to the new URL" | "Temporarily elsewhere, keep using the old URL" |
| **Browser caching** | **Caches the redirect** — remembers mapping, skips server next time | **Does NOT cache** — asks server every time |
| SEO | Search engines transfer ranking to the new URL | Ranking stays with the original URL |
| Use case | Permanent moves: old domain → new, http → https | A/B tests, maintenance pages, click-tracking |
| Downside | Can't track clicks (browser bypasses after caching) | More server load (every click hits you) |

> **One-liner:** 301 = "forever, and remember it" (cached). 302 = "for now, ask me each time" (not cached). This caching difference is exactly why a 301 TinyURL disappears from the Network tab on repeat visits.

---

## Why 301 Is in "All" but NOT in "Fetch/XHR" (DevTools)

DevTools Network filter tabs (All, Fetch/XHR, Doc, CSS, JS, Img…) group requests by **how they were made (request type / initiator)** — NOT by status code (301/302/200).

### What "Fetch/XHR" means
Shows **only** requests made *programmatically from JavaScript* via:
- `fetch(...)` API
- `XMLHttpRequest` (XHR) — i.e. AJAX calls

### What a TinyURL request is
Clicking/pasting `tinyurl.com/...` is a **top-level page navigation** — loading a whole *document*, not a JS `fetch()`. So DevTools files it as a **`Doc`** request, not Fetch/XHR.

```
DevTools Network filters = categorize by REQUEST TYPE (how it originated):

  All        → everything (Doc + Fetch/XHR + JS + CSS + Img + Font ...)
  Doc        → page navigations (HTML documents)   ← the 301 redirect lives HERE
  Fetch/XHR  → only fetch()/XMLHttpRequest from JS  ← the redirect is NOT here
  JS / CSS   → script & stylesheet files
  Img        → images
```

- **"All"** = every request regardless of type → 301 shows up ✓
- **"Fetch/XHR"** = only JS-initiated calls → a navigation isn't one → 301 absent ✗

### Key insight
```
The filter is NOT about status code (301/302/200).
It's about HOW the request was made:

  - Typed/clicked a URL → navigation → "Doc"
  - JS called fetch()    → "Fetch/XHR"
```

A 301 *could* appear under Fetch/XHR — but **only if** JavaScript did `fetch('https://tinyurl.com/...')` and the server replied 301. Since the redirect came from navigating directly, it's under **Doc** (and "All"), never Fetch/XHR.

**Quick check:** click the **"Doc"** filter → the `301` is there. Click **"Fetch/XHR"** → it's gone. Confirms it's a document navigation, not an AJAX call.

---

## Real Worked Example: Response Body & Where the Long URL Lives

Real request: `https://tinyurl.com/56j77t25` → Wikipedia's Elon Musk page.

### The response body
A redirect *does* include a small HTML body — a fallback:

```html
<!DOCTYPE html>
<html>
    <head>
        <meta charset="UTF-8" />
        <meta http-equiv="refresh" content="0;url='https://en.wikipedia.org/wiki/Elon_Musk'" />
        <title>Redirecting to https://en.wikipedia.org/wiki/Elon_Musk</title>
    </head>
    <body>
        Redirecting to <a href="https://en.wikipedia.org/wiki/Elon_Musk">https://en.wikipedia.org/wiki/Elon_Musk</a>.
    </body>
</html>
```

**What it's for — a fallback.** The browser normally redirects via the **`Location` header**. The body is a safety net using two backups:
- `<meta http-equiv="refresh" content="0;url=...">` — meta refresh, in case the header is ignored.
- A clickable `<a href>` — so a human/old browser can click manually.

You normally **never see this page** — the `Location` header fires instantly. It's just a backup.

### Does the long URL come in the REQUEST or the RESPONSE?
**The long URL comes back in the RESPONSE — never in the request.**

```
───────────── REQUEST (browser → tinyurl) ─────────────
GET /56j77t25 HTTP/2
Host: tinyurl.com
        ↑
   ONLY the short code. The browser does NOT know the long URL yet.
   It has to ASK the server.

───────────── RESPONSE (tinyurl → browser) ────────────
HTTP/2 301
location: https://en.wikipedia.org/wiki/Elon_Musk     ← long URL HERE
...
<body> Redirecting to https://en.wikipedia.org/... </body>  ← and HERE
        ↑
   Server LOOKED UP 56j77t25 in its DB, found the long URL,
   and SENT IT BACK in the response.
```

The browser only has the short code — it has no idea where it points. The **server holds the mapping** (`56j77t25 → wikipedia.org/.../Elon_Musk`) in its DB, looks it up, and returns the long URL.

### The long URL appeared 4 times in this real response
| Where | Purpose |
|---|---|
| **`location:` header** | The real redirect mechanism (only this one matters) |
| `x-lighttpd-longurl:` header | TinyURL custom/debug header |
| `x-tinyurl-target:` header | Another TinyURL custom header |
| **Body** (`<meta refresh>` + `<a href>`) | HTML fallback page |

### Other things the real response revealed
- **`HTTP/2 301`** — permanent redirect (browser may cache → can vanish from Network tab).
- **`cache-control: no-cache, no-store, private`** — TinyURL tells the browser *not* to cache despite the 301, so it can keep tracking clicks.
- **`server: cloudflare`, `cf-ray`, `cf-cache-status`** — TinyURL runs behind **Cloudflare** (CDN/proxy).
- **`set-cookie: __cf_bm`** — Cloudflare's bot-management cookie.

### Summary
- **Body** = a small HTML fallback page (`meta refresh` + clickable link), used only if the `Location` header isn't honored.
- **Long URL location** = the **response** (`Location` header + body), **not** the request. The request carries only the short code; the server looks it up and returns the long URL.
