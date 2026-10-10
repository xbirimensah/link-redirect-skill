# link-redirect

An agent skill (Claude Code compatible) that builds a link redirect page: tap a
link inside Instagram, Facebook or TikTok and it opens the end goal link in its
native app (YouTube, Spotify, Instagram, WhatsApp, ...) or the default handler,
instead of the in-app browser. Works for any destination URL.

- Android: `intent://` hand-off with web fallback
- iOS: app URL scheme with timed web fallback
- Desktop: normal redirect
- One destination per page, or many via `?u=` with an allowlist
- Single static `index.html`, no build step, host anywhere

## Install

```bash
git clone https://github.com/xbirimensah/link-redirect-skill ~/.claude/skills/link-redirect
```

Then ask your agent: "create a link redirect to https://example.com/my-page that
opens in the app instead of the in-app browser, on go.example.com".

## Use without an agent

1. Copy `template/index.html`.
2. Set `defaultTarget` (and optionally `allowedHosts`) in the `CONFIG` block.
3. Deploy it to any static host and point a hostname at it.

See `SKILL.md` for the full workflow, how to add apps, verification status and gotchas.

## License

MIT
