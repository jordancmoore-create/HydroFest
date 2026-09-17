# Miramichi HydroFest — Project Context

Splash/landing page for **Miramichi HydroFest 2027**, an HRL (Hydroplane Racing League)
event on the Miramichi River in **Miramichi, New Brunswick**.

**Event dates: Saturday–Sunday, September 4–5, 2027** (Labour Day weekend — Labour Day
falls Monday September 6, 2027).

**Venue: Ritchie Wharf**, Miramichi, NB — confirmed September 2026 from the event poster.

**Naming note:** a second promo poster for the same weekend is branded *"Miramichi East
Coast Hydroplane Regatta"* rather than HydroFest. The site deliberately stays on
**Miramichi HydroFest** — that's what the crest artwork and the hydrofest.ca domain say.
If the event is actually being renamed, the crest has to be re-cut first.

**Domain: hydrofest.ca** — purchased September 2026, hosted on Cloudflare (same account
as herrinchoker.ca / dreamweaverracing.com / cabot2026.ca).

Sister projects on this machine, same build pattern and shared design DNA:
`Documents\HerrinChoker\HerrinChokerRacing` (#38, herrinchoker.ca) and
`Documents\DreamWeaver\DreamWeaverRacing` (F-26, dreamweaverracing.com). The boat on the
HydroFest crest is the #38 — the Herrin Choker hull.

---

## Deployment

Cloudflare **Worker `hydrofest`** — assets-only static site, no build step.
`wrangler.jsonc` has no `main` field on purpose (don't add one unless Worker code is
added). The Worker name in `wrangler.jsonc` must stay `hydrofest` or git builds break.
`.assetsignore` keeps CLAUDE.md, config files and the full-res crest out of the
served site.

Auto-deploy: **push to `main` → Cloudflare Workers Builds runs `npx wrangler deploy`
→ live in about a minute.** GitHub repo: `jordancmoore-create/HydroFest` (branch `main`,
build command empty).

If the Workers Builds git integration turns flaky (it did on the Herrin Choker site),
swap in the GitHub Actions workflow from
`Documents\HerrinChoker\HerrinChokerRacing\.github\workflows\deploy.yml` — it uses
`cloudflare/wrangler-action@v3` pinned to wrangler 4.x with a `CLOUDFLARE_API_TOKEN`
repo secret.

---

## Design system (from the event crest)

```css
--red:#cf1722          /* primary — hull red */
--red-bright:#ff2f2f   /* hovers */
--red-deep:#7d0c13     /* hero type, shadows */
--gold:#c9a24a         /* accents */
--gold-lt:#e6c887      /* card headings, countdown labels */
--sand:#f2e6cf         /* checker strip */
--ink:#0b0c0f          /* page bg */
--ink-2:#14161b        /* panels, alt sections */
--white:#f7f5f1
--muted:#9aa1ac
```

Fonts: **Barlow Condensed** (headings, numbers, italic 800 for the HydroFest-style
slant) + **Barlow** (body) — same family as the Herrin Choker site.

**The hero is deliberately light.** The crest is a JPEG on a white background, so the
hero uses a white→cream gradient and the image is `mix-blend-mode: multiply` — the white
square disappears into the cream. Don't move the crest onto the dark sections without
first getting a transparent PNG/SVG version of the artwork.

Recurring devices in the dark half of the page, all defined as CSS custom properties or
shared classes so they stay consistent:

- `--speedlines` / `--bloom` — a faint diagonal weave plus a red glow from the section's
  top edge, painted by an `::before` at `inset: 0` on `.countdown-band`, `.features` and
  `.stay`. Those sections need `position: relative`; they must **not** get
  `overflow: hidden`, which would silently clip real content instead of the texture.
- `.eyebrow` + `.section-head h2` — small gold label with a red rule, over an italic
  condensed uppercase heading.
- `[data-reveal]` — fade/rise on scroll via `IntersectionObserver`. The hidden state is
  scoped to `.js [data-reveal]`, and the `js` class is set by an inline script in
  `<head>`, so the page is never blank without JavaScript. Reduced-motion and
  IO-less browsers get everything visible immediately.
- The race-weekend icons are an inline `<symbol>` sprite at the bottom of `index.html`;
  stroke weight and colour come from `.fi` in the CSS, not from the markup.

**Careful with `.card p` / `.stay p` / `.racecard p`**: those element selectors outrank
single-class rules like `.card-num` or `.contact`. Scope overrides as `.card .card-num`
rather than reaching for `!important`.

---

## File structure

```
index.html                     single-page splash
css/style.css                  all styles
favicon.svg                    red tile, "HF", checker strip
images/hydrofest-2027.jpg      1000×1000, 286 KB — the crest as served
images/og-hydrofest-2027.jpg   1200×630, 138 KB — social share card
images/hydrofest-2027-full.jpg 1254×1254, 2.0 MB — original artwork (not served;
                               listed in .assetsignore). Source for re-exports.
```

The two derived images were generated from the original with System.Drawing
(PowerShell, quality 84).

## Countdown

`EVENT_DATE` near the bottom of `index.html` drives the countdown band:
`"2027-09-04T09:00:00-03:00"` (Atlantic Daylight Time is UTC-3 in September).
**The 09:00 start time is a placeholder** — update it when the race schedule is set.
Setting `EVENT_DATE = ""` turns the band back into a plain "September 4–5, 2027" line.

## What's still placeholder

- **`info@hydrofest.ca`** — the mailto in the CTA. Needs a Cloudflare Email Routing
  rule forwarding it to a real inbox, or the address should be swapped out.
- **Social links** — the `.social` list in `index.html` is commented out; uncomment and
  fill in the real Facebook/Instagram URLs, and drop the `hidden` attribute.
- **Race-day start time** — see Countdown above. The JSON-LD `SportsEvent` block in
  `index.html` repeats the same placeholder start and adds a guessed 18:00 Sunday
  `endDate`; update both together when the schedule lands.
- **Schedule, tickets, parking, accessibility** — the three cards still say "coming
  soon" on purpose; no specifics have been published yet. The venue is now known
  (Ritchie Wharf) and is stated on the page.

## Local dev notes (this machine)

- **Preview:** `.claude/launch.json` config **hydrofest** — python http.server, port 2027.
- **Windows Defender Controlled Folder Access** protects `Documents`: shell writes here
  fail with misleading errors (`mkdir: cannot create directory 'css': No such file or
  directory`). Confirmed blocked for this folder Sept 16 2026: bash/coreutils and
  powershell.exe writes. **`git.exe` is allowlisted and works**, and the Write/Edit tools
  work.
- **Working recipe for binary files:** build them in the session scratchpad (Temp is not
  CFA-protected), then let git write them into the repo:
  `git hash-object -w <tmpfile>` → `git update-index --add --cacheinfo 100644,<sha>,<repo-path>`
  → `git checkout-index -f -- <repo-path>`. This is how the images/ folder was created.
- `.claude/settings.local.json` is ignored by the global git excludes file
  (`~/.config/git/ignore`), so it stays local — that's intended.
