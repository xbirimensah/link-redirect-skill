---
name: link-redirect
description: Build and deploy a link redirect page that breaks out of in-app browsers (Instagram, Facebook, TikTok, etc.) and opens a native app such as YouTube instead. Use when the user wants a link that opens an app rather than the in-app browser, a "deep link" or "app redirect" page, or a short redirect on their own domain.
---

# Link redirect (open the app instead of the in-app browser)

When someone taps a link inside Instagram, Facebook or TikTok, it opens in that
app's in-app browser. This skill builds a tiny static page that detects the
platform and hands the link to the native app instead, with a web fallback.

## How it works

| Platform | Mechanism | Reliability |
|---|---|---|
| Android | `intent://` URL with the target app's package name and a `browser_fallback_url` | High |
| iOS | The app's URL scheme (e.g. `youtube://`), then a timed fallback to the https URL | Good, depends on the in-app browser |
| Desktop | Plain redirect to the https URL | Always |

The page also shows an "Open in app" button, because some iOS in-app browsers
only allow the hand-off from a direct tap.

## Workflow

1. **Collect inputs from the user** (ask only for what is missing):
   - Destination URL (e.g. a channel, video, or profile).
   - Target app: Android package name and iOS URL scheme (see table below).
   - Public hostname the link should live on (e.g. `go.example.com`), and who hosts its DNS.
2. **Copy the template.** Copy `template/index.html` into a new, empty project directory.
3. **Edit only the `CONFIG` block** at the top of the script: `defaultTarget`,
   `allowedHosts`, `androidPackage`, `iosScheme`, and `normalize` if the app has
   short links. The defaults are set up for YouTube.
4. **Deploy as a static site.** Any static host works:
   - Vercel: `vercel deploy --prod --yes` from the project directory.
   - Netlify: `netlify deploy --prod --dir .`
   - Cloudflare Pages: `wrangler pages deploy .`
   - GitHub Pages: push `index.html` and enable Pages.
5. **Attach the hostname** in the host's dashboard or CLI, then create the DNS record it asks for.
6. **Test on a real phone** by sending the link to yourself in an Instagram DM and tapping it.

## Usage after deploy

- `https://<host>/` opens the configured default destination.
- `https://<host>/?u=<https URL>` opens a specific destination. Only hosts in
  `allowedHosts` are accepted, so the page cannot be abused as an open redirect.

## Known app values

Verified means tested end to end. Others are starting points to test before relying on them.

| App | Android package | iOS scheme | Status |
|---|---|---|---|
| YouTube | `com.google.android.youtube` | `youtube://` | Verified |
| Spotify | `com.spotify.music` | `spotify://` | Untested, URL mapping may need a custom `normalize` |
| Instagram | `com.instagram.android` | `instagram://` | Untested, profile links need `user?username=` mapping |
| X / Twitter | `com.twitter.android` | `twitter://` | Untested |
| WhatsApp | `com.whatsapp` | `whatsapp://` | Untested |

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
- **Keep `allowedHosts` tight.** Never accept arbitrary `?u=` values.
