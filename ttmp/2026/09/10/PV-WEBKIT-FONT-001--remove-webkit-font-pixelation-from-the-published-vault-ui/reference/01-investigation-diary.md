---
Title: Investigation diary
Ticket: PV-WEBKIT-FONT-001
Status: active
Topics:
    - frontend
    - styling
    - ux
    - regression
DocType: reference
Intent: long-term
Owners: []
RelatedFiles:
    - Path: repo://publish-vault/web/src/styles/base.css
      Note: Global WebKit font smoothing and image-rendering declarations under investigation.
    - Path: repo://publish-vault/web/src/index.css
      Note: Frontend stylesheet entry point that imports base.css.
ExternalSources: []
Summary: 'Chronological implementation diary for removing WebKit font pixelation from the published vault UI.'
LastUpdated: 2026-09-10T16:20:00-04:00
WhatFor: Resuming or reviewing the WebKit font-rendering fix.
WhenToUse: Read before continuing work on PV-WEBKIT-FONT-001.
---

# Investigation diary

## Goal

Remove the unintended pixelated font rendering in WebKit browsers while preserving the application's intentional retro visual treatment. Confirm the smallest CSS-only change, validate the frontend build and type checks, and leave the ticket with an evidence-backed implementation record.

## Step 1: Create the ticket and locate the rendering control

The ticket workspace was created before changing source files so the scope, tasks, and investigation record have a stable home. I then traced the stylesheet entry point into the global base rules and searched the repository for font-rendering and pixelation declarations. The issue is localized: `web/src/index.css` imports `web/src/styles/base.css`, and `html` in `base.css` explicitly disables WebKit font smoothing with `-webkit-font-smoothing: none`.

The same rule also sets `font-smooth: never` and `image-rendering: pixelated`. The latter concerns raster/image rendering rather than glyph antialiasing, so it should not be changed as part of a narrowly scoped font fix. The likely one-line correction is to replace the WebKit value `none` with `antialiased`, allowing WebKit to smooth glyph edges while retaining the retro palette, fonts, sizes, and layout.

### Prompt Context
**User prompt (verbatim):** "Create a new docmgr ticket to remove the font pixelation on webkit. I think it's just a simple css tweak. commit at appropriate intervals and keep a detailed diary as you work (using the diary format from the skill)"
**Assistant interpretation:** Create a new docmgr ticket, investigate and implement the smallest CSS fix for WebKit font pixelation, validate it, commit coherent milestones, and maintain a detailed investigation diary in the ticket.
**Inferred user intent:** Make text readable in WebKit without undoing the site's intentional retro styling.
**Commit (code):** Not yet; investigation checkpoint only.

### What I did
- Ran `docmgr status --summary-only` to confirm the repository's existing docmgr workspace and vocabulary.
- Created ticket `PV-WEBKIT-FONT-001` with topics `frontend`, `styling`, `ux`, and `regression`.
- Added the ticket diary and three focused tasks.
- Read `web/src/index.css` and `web/src/styles/base.css`.
- Searched the source tree for `font`, `webkit`, `text-render`, `antialias`, `pixel`, and `image-render` declarations.
- Confirmed the stylesheet import order: `index.css` imports `tokens.css`, `bridge.css`, `base.css`, `chrome.css`, and `prose.css`.

### Why
- A global `-webkit-font-smoothing: none` declaration directly requests unsmoothed, pixelated glyph rendering in WebKit.
- Changing only that declaration minimizes visual and behavioral risk. It avoids conflating font smoothing with `image-rendering: pixelated`, which is a separate raster-image policy.
- `antialiased` is supported by WebKit and explicitly requests grayscale antialiasing, removing the reported glyph pixelation without introducing a font dependency.

### What worked
- The repository search found one global WebKit font-smoothing declaration in `web/src/styles/base.css`, making the proposed change easy to isolate.
- The relevant stylesheet is part of the normal Vite entry path, so a CSS edit will be covered by the existing frontend build.

### What didn't work
- No failures observed during ticket creation or investigation.

### What I learned
- The retro design notes intentionally mention pixel-crisp rendering, but the current implementation applies that choice globally to all text through the `html` rule. The requested fix should narrow that policy rather than remove the broader visual language.
- `font-smooth: never` is non-standard and does not provide the WebKit-specific control needed for this issue; `-webkit-font-smoothing` is the actionable declaration.

### What was tricky to build
- Separating glyph rasterization from image rasterization: both are described with “pixelated” language, but only `-webkit-font-smoothing: none` controls the WebKit font symptom. Touching `image-rendering` would broaden the change unnecessarily.

### What warrants a second pair of eyes
- Review whether grayscale `antialiased` is the desired WebKit appearance versus omitting the vendor declaration and accepting the browser default. The former is deterministic and directly reverses the current `none` policy; the latter would vary by browser/platform.
- Confirm that the global rule should remain global and that no component intentionally depends on unsmoothed text.

### What should be done in the future
- Validate the final appearance in a WebKit-capable browser on the affected deployment or a representative local page. Automated CSS/build checks cannot prove visual glyph quality.

### Code review instructions
- Review `web/src/styles/base.css`, the `html` rule near the top of the file.
- Confirm `web/src/index.css` still imports `base.css` in the expected order.
- Run `pnpm --dir web check` and `pnpm --dir web build`.

### Technical details
- Current declaration: `-webkit-font-smoothing: none`.
- Proposed declaration: `-webkit-font-smoothing: antialiased`.
- Deliberately unchanged: `font-smooth: never`, `image-rendering: pixelated`, font-family stacks, font sizes, weights, and all component styles.
