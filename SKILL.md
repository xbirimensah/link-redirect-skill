---
name: link-redirect
description: Build and deploy a link redirect page that breaks out of in-app browsers (Instagram, Facebook, TikTok, etc.) and opens the end goal link in its native app (YouTube, Spotify, Instagram, WhatsApp, ...) or the default handler. Works for any destination URL. Use when the user wants a link that opens an app rather than the in-app browser, an "app redirect" or "deep link" page, or a redirect on their own domain.
---

# Link redirect (open the app instead of the in-app browser)

When someone taps a link inside Instagram, Facebook or TikTok, it opens in that
app's in-app browser. This skill builds a tiny static page that detects the
platform and hands the link to the native app instead, with a web fallback.
It works for any destination URL.

## How it works

| Platform | Known app (host in `APPS`) | Any other https link |
|---|---|---|
| Android | `intent://` with the app's package name | `intent://` without a package: opens the default handler, out of the in-app browser |
| iOS | App URL scheme, then timed fallback to https | Plain https (universal link) |
| Desktop | Plain redirect | Plain redirect |

The page also shows an "Open in app" button, because some iOS in-app browsers
only allow the hand-off from a direct tap.

## Workflow

1. **Collect inputs from the user** (ask only for what is missing):
   - The end goal link (any https URL).
   - Whether one page should serve several destinations (`?u=`) or just one.
   - The hostname the link should live on (e.g. `go.example.com`) and who hosts its DNS.
2. **Copy the template.** Copy `template/index.html` into a new, empty project directory.
3. **Edit only the `CONFIG` block** (and `APPS` if the destination's app is missing):
   - `defaultTarget`: the end goal link.
   - `allowedHosts`: `[]` (only the default target's host), a list of extra hosts,
     or `["*"]` for any host. `["*"]` makes the page an open redirect: anyone can
     craft a link on the user's domain that goes anywhere. Warn the user and
     default to `[]`.
4. **Deploy as a static site.** Any static host works:
   - Vercel: `vercel deploy --prod --yes` from the project directory.
   - Netlify: `netlify deploy --prod --dir .`
   - Cloudflare Pages: `wrangler pages deploy .`
   - GitHub Pages: push `index.html` and enable Pages.
5. **Attach the hostname** in the host's dashboard or CLI, then create the DNS record it asks for.
6. **Test on a real phone** by sending the link to yourself in an Instagram DM and tapping it.

## Usage after deploy

- `https://<host>/` opens `defaultTarget`.
- `https://<host>/?u=<https URL>` opens a specific destination, if its host is allowed.

## Adding an app

Append an entry to `APPS`:

```js
{ hosts: ["example.com", "www.example.com"],
  android: "com.example.app",                       // Android package name
  ios: function (u) { return "exampleapp://" + u.host + u.pathname; } }  // or null to use https
```

Find an Android package name in the app's Play Store URL (`?id=...`). iOS URL
schemes are app-specific: check the app's docs or test the scheme in Safari.

## Verification status

| Destination | Status |
|---|---|
| YouTube (Android intent, iOS scheme, web fallback) | Verified on iPhone |
| Spotify, Instagram, X, WhatsApp, Telegram | Untested, test before relying on them |
| TikTok, LinkedIn, Facebook | Android package only; iOS uses https, untested |
| Any other https link | Package-less Android intent and https on iOS, untested per site |

## Gotchas

- **DNS proxy:** if the DNS is on Cloudflare, create the record as **DNS only**
  (grey cloud) when the host terminates TLS itself (Vercel, Netlify). A proxied
  record causes HTTP 525 errors.
- **Certificate:** if HTTPS fails right after attaching the domain, request the
  certificate explicitly (on Vercel: `vercel certs issue <host>`).
- **Local DNS cache:** after changing DNS, your own machine may keep the old
  answer for a few minutes. Test with `curl --resolve <host>:443:<ip>` or from a phone.
- **iOS in-app browsers differ.** If the scheme is blocked, the user lands on the
  web fallback after the timeout, so the link still works, just not in the app.
- **Unknown iOS destinations** open as https, so the in-app browser may stay
  open. Add an `APPS` entry with the app's scheme to fix that.
