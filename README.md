# link-redirect

An agent skill (Claude Code compatible) that builds a link redirect page: tap a
link inside Instagram, Facebook or TikTok and it opens the native app (YouTube
by default) instead of the in-app browser.

- Android: `intent://` hand-off with web fallback
- iOS: app URL scheme with timed web fallback
- Desktop: normal redirect
- Single static `index.html`, no build step, host anywhere

## Install

```bash
git clone https://github.com/xbirimensah/link-redirect-skill ~/.claude/skills/link-redirect
```

Then ask your agent: "create a link redirect that opens my YouTube channel
instead of the in-app browser on go.example.com".

## Use without an agent

1. Copy `template/index.html`.
2. Edit the `CONFIG` block at the top of the script.
3. Deploy it to any static host and point a hostname at it.

See `SKILL.md` for the full workflow, known app values and gotchas.

## License

MIT
