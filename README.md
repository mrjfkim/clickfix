# ClickFix Test Bed

A live clipboard-hijack test page for demonstrating clipboard-protection tooling against
[ClickFix](https://en.wikipedia.org/wiki/ClickFix)-style social engineering, in a real Chromium
browser.

## What it does

ClickFix lures trick a visitor into opening the Windows Run dialog (or a terminal) and pasting a
command copied from a fake CAPTCHA, error, or update prompt. This page reproduces four common lure
patterns and, for each one, performs a **real** `navigator.clipboard.writeText()` call (with a
legacy `execCommand('copy')` fallback) when you click "Copy fix command" — the same delivery
mechanism real ClickFix payloads use. That gives a browser extension or clipboard-security tool
something genuine to intercept, so you can demo it live instead of scripting a fake result.

A status log under each scenario polls the clipboard after the write and lets you re-check it on
demand, so a quarantine-and-scan flow (write → placeholder → verdict) is visible as it resolves.

## What it does *not* do

- It never executes anything on your machine. Browsers can't run local system commands from a
  webpage — the "Run" dialog on the page is a visual mockup. Only a human manually pasting into a
  real Run/terminal window and pressing Enter could execute the payload.
- It makes no network requests and loads no external resources — everything is inline in
  `index.html`.
- Every payload domain is defanged (`hxxps://`, `[.]invalid`) and non-resolvable, so even if a
  payload were somehow copied out of this page and pasted into a real terminal, it would fail
  immediately rather than do anything.
- The lure UI is intentionally generic — it does not reproduce any real company's logo or
  wordmark, even though real ClickFix campaigns often spoof recognizable brands.

## Demoing both protection modes

The page's behavior is identical regardless of configuration: it always attempts the same real
clipboard write. What differs is what your extension does with that write:

- **Clipboard API Only** — blocks every programmatic clipboard write outright. Expect the write
  attempt to fail immediately for every scenario on this page.
- **AI Classifier** — lets the write land, quarantines the clipboard behind a placeholder while it
  scans, then restores or blocks based on the verdict. Use "Check clipboard now" right after
  copying, and again a few seconds later, to watch the placeholder resolve.

## Hosting on GitHub Pages

1. Push this folder to a repository.
2. In the repo, go to **Settings → Pages**, set **Source** to the branch containing `index.html`
   (root folder).
3. GitHub serves the page over HTTPS automatically — required, since `navigator.clipboard` only
   works in a secure context.

`<meta name="robots" content="noindex, nofollow">` is set so the page doesn't get indexed by search
engines.

## Local testing

Any static file server works, e.g.:

```bash
npx serve .
```

Opening `index.html` directly via `file://` may also work in Chromium, but a local server is more
representative of the real HTTPS deployment.
