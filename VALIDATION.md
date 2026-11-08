# Validation record

## Phone entry and achievement revision (2026-11-08)

This revision addresses only findings `d6e8b9dfbe47b8bdbcb2c9c720942838eaed40b2ff72190c90145e6d5c0dacdf` and `bad89573dc88dfab47ecbbcf20313577a6778705901265646ceec2fcea5e6981`. Both reproduced before source edits. The existing game, feed rules, scoring, save format, assets, dependencies and site/domain metadata are preserved. Answers are in `.imd-responses.json`.

### Reproduction, fixes and rechecks

The supplied Chromium browser rendered production assets with `test/browser.mjs`'s schema-valid `mockedData` fixtures: installation `sepolia-browser-test-only`, time `2026-11-08T12:05:00Z`, open round 101 closing at 12:10. Each initial entry used fresh local storage, a loaded Outfit font, a 390 × 844 viewport and `scrollY=0`.

| Finding | Before: reproduced | Source fix and final measurement |
| --- | --- | --- |
| High: invitation hides the main action | Invitation x=16, y=254.671875, w=358, h=121. Enabled lock y=853.25–899.25, outside the screen | `src/App.tsx:504`, `src/style.css:1779`: only phone Live Play with an undismissed invitation visually omits the intro and its space. The heading remains accessible through a clipped 1px element. Final invitation y=126–247; enabled lock y=724.578125–770.578125; deadline ends at y=795.578125. The complete round, friend call and editable-link/local-only disclosure remain. Ordinary entry still has its headline and lock y=712.25–758.25. Practice, other views, desktop and dismissal restore the normal intro |
| Medium: First Call blocks Duel a friend | Share y=772.25–814.25. Toast x=12, y=772, w=232, h=60. `elementFromPoint(195,793.25)` returned `.toast`; activating that hit target did not open the link | `src/App.tsx:450`, `src/App.tsx:889`, `src/style.css:2291`: the existing status notice follows the controls; at ≤560px it is static with 12px spacing when populated. Final share position is unchanged; toast x=33, y=852.25, w=324, h=60. The share centre hits the button. A supplied-browser click opens its selectable link with First Call still present and no scroll. Dismissal closes 72px of document space above the Professor. `src/App.tsx:619` retains the notice above content in other views. At 561px and larger the existing fixed toast remains |

A first implementation used `display:none` for the intro. Axe then reported `page-has-heading-one`; preserving the semantic heading with the visually hidden pattern fixed that finding without restoring hero space. The final friend entry passes all 44 applicable axe checks.

`test/browser.mjs:348` adds share/notice geometry and raw touchscreen assertions before dismissal. Its friend-entry checks also cover the visible lock/deadline, complete disclosure and responsive intro. The new assertions are delivered for a normal Playwright runtime; the shell suite could not reach them here, as explained below.

### Commands and actual outcomes

Dependencies were installed only in disposable `test/scratch/check/`, from byte-identical copies of the existing manifest and lockfile. Source, public assets and configuration were mirrored there for checks; final source/configuration were compared byte for byte against that successful build copy. No root dependency directory, dependency archive or manifest change is delivered.

| Command or check | Actual result |
| --- | --- |
| `npm ci --prefix test/scratch/check --cache test/scratch/npm-cache --no-audit --no-fund` | Passed; installed 83 packages from the existing lockfile |
| `npm run typecheck --prefix test/scratch/check` | Passed, no diagnostics on final source |
| `npm test --prefix test/scratch/check` | **61 passed, 0 failed** |
| `npm run build --prefix test/scratch/check` | Passed after the final source change; Vite 6.4.1, 1778 modules. Its complete export was copied to root `dist/`: `index-CHmCh6ug.js`, `index-h4Xec68w.css`, unchanged `api-schemas-BuEuZ2Un.js` and local runtime assets |
| `node --check test/browser.mjs` | Passed |
| `npm run test:browser --prefix test/scratch/check`, with the supplied Playwright module/browser paths | **Could not run interactions**: Chromium exited with SIGTRAP during launch; command exited 1. This is a runtime limitation, not a passing suite |
| `node .imd/reads/skills/design-critic/scripts/taste.mjs --out test/scratch/taste`, with the supplied Playwright module/browser paths | Attempted; reported a failed `render` check because Chromium could not launch. **No 16/16 probe result is claimed for this revision** |

### Supplied-browser coverage and transport limitation

The supplied browser works, but could not reach the shell's local HTTP preview. For rendered validation, a disposable HTTPS document loaded the final `dist/` JavaScript, CSS, font, favicon and schema chunk in memory. Only asset transport URLs were embedded; fixture fetch responses and the clock were isolated test overrides. This exercised the exported application and reproduced both reports' exact original rectangles. It does **not** establish local HTTP/subpath delivery, live hosting headers, independent fresh browser contexts or raw touchscreen input. Initial sender and recipient entries instead used separate document loads with cleared local storage. The normal HTTP/Playwright regressions remain in `test/browser.mjs`.

Actual supplied-browser interaction checks:

- The 390px friend entry retained its complete invitation and deadline above the fold. Space/Tab/Enter selected Sprint, bracket 1–10 and locked the call; the lock's 3px focus ring was visually inspected. Saved data retained the friend's Nap and stayed pending before settlement. Reloading with the confirmed settlement fixture at 12:11 resolved **30 player points vs 0 friend points** and unlocked Duelist. The result dialog closed with Escape.
- With First Call open, the ordinary phone share button remained unobscured and opened the selectable link without scrolling. Dismissal removed the notice and its space. Awards retained a single dismissible notice above content. Switching Practice/Live and dismissing the invitation restored the appropriate intro. The three fixed practice rounds completed for **90 practice points**, three practice results and Hat Trick, leaving the **30 live points** separate.
- Ordinary and friend entry were measured at **320/390/1440px** using the pinned probe's unmodified DOM `collect()` function through the supplied browser. Both samples had no horizontal overflow, text overlaps, floating-control obstruction, clipped text or crowded targets below the 24px/spacing bar. Measured text met 4.5:1 (3:1 large). Eight text boxes per paired phone/desktop sample had an unmeasurable background; they are not counted as contrast passes. The interface retained Outfit, three weights and at most six repeated sizes. Phone prose counts were 22 words for ordinary entry and 36 for friend entry. This supplementary DOM measurement is not a successful run of the full probe command.
- Widths **560/561/850px** confirmed the exact phone breakpoint and no horizontal overflow. At 560px the notice was static and the friend intro occupied 1px outside flow; at 561px and above the intro and fixed toast returned. Reduced-motion emulation on the actual app left **zero running infinite animations**; Pause changed to Resume. Tested application states recorded no uncaught JavaScript errors.

Axe results on the final export: **44 checks / zero violations** for phone friend entry; **44 / zero** for the confirmed friend board with its Duelist notice; **19 / zero** for the friend result dialog. These scans and keyboard checks are not a human screen-reader session or a claim of full accessibility compliance.

Screenshots were made and inspected for both original defects and final 390px entry/notice states, desktop, keyboard focus and result. Disposable evidence paths include `test/scratch/browser/before-friend-entry-390.png`, `before-first-call-390.png`, `after-friend-entry-390.png`, `after-first-call-390.png`, `normal-final-{320,390,1440}.png`, `friend-final-{320,390,1440}.png`, `friend-result-final-390.png` and `practice-final-390.png`. Scratch screenshots, test dependencies and browser-generated logs are not delivery assets.

### Better Interface consolidated review

| Domain | Coverage, finding and resolution |
| --- | --- |
| Accessibility — Checked | Existing native controls, names, status region and dismiss control retained. Keyboard locking, visible focus, Escape, reduced motion and scoped axe scans checked. Fixed the missing semantic heading introduced by hiding the hero. Human screen readers, forced colors and native zoom remain unverified |
| Layout — Checked | Reproduced and fixed both root causes above. Final 390px screenshots inspected; DOM measurements at 320/390/1440 and breakpoint checks at 560/561/850. Professor movement measured in document coordinates, accounting for scroll. RTL and browser-native 200% zoom unverified |
| Writing — Checked | Existing action labels, friend call/round, deadline and editable/local-only disclosure remain intact. No new promotional copy or timing/result claims; pending and confirmed states exercised |
| Typography — Checked | Existing local Outfit file, six-role scale and weights retained. Font loaded in the browser; ordinary/friend text wrapped without clipping in measured widths. No new size or weight added |
| Colors — Checked | Existing semantic hex tokens preserved. Rendered solid-background pairs passed the pinned collector thresholds and scoped axe audits. Image/canvas/background pairs excluded by the collector remain unverified; no new theme was added |
| UI — Checked | Live entry, selected, locked, share, notice, dismiss, result, Practice and Awards fallback states exercised. Existing notice surface, icons, queue, manual dismiss and desktop position preserved. Motion source guards and reduced-motion/Pause behavior checked; animation-panel replay at 10% speed not performed |

Complete for the two reported defects and the available rendered checks. The standalone browser suite/probe, HTTP/subpath browser preview, WebKit/native Safari, human screen readers, RTL and native 200% zoom remain unverified in this run. Earlier broader/live-feed validation below is historical and was not re-established by this revision. IMD publishes the delivered `dist/` to the existing site/domain after review; this worker performed no publishing action.

Separate hosting observation: direct inspection of the existing live domain reported an Ajv `new Function` CSP error. A `curl -I` check confirmed its response sets `script-src 'self' 'unsafe-inline' 'wasm-unsafe-eval'`, without allowing that compilation. This is the current hosting response policy, absent from the repository, and needs IMD/the hosting owner to settle or recheck at handoff. The two requested phone fixes do not change the accepted schema-validation implementation. The isolated browser checks above do not establish that the live domain is usable under that policy.

Final integrity audit: the export contains **9 files / 659,146 bytes**. The **47 eligible project files total less than 1.5 MiB**, below the 8 MiB budget. HTML/CSS/module local references resolve and use relative asset URLs; title, favicon, description, `og:title` and `og:image` remain present for the existing domain. Final source and build-copy configuration match byte for byte. The manifest, lockfile and 55-byte ignore file are unchanged; `dist/` is included. No source map, package archive, symlink, dependency/cache directory or browser-generated log is in the submission set. `.imd-responses.json` was parsed and checked against both exact finding IDs.

---

## Update build: continuation of job 3f8539ac-d327-494f-9f07-acf232e54e08 (2026-11-08)

The tree from https://github.com/identity-md-launches/launch-3134-build-publish-squeek-beat-the-ai, as served at https://squeek-beat-the-ai-predictor.imd.site/, was restored unchanged. Every check was re-run on it before any edit (typecheck clean, 61/61 unit tests, all Chromium flows, probe 16/16). After that, a Better Interface review led to the fixes below, and the export was rebuilt after the last source change. These are checks the worker ran, not an independent certification.

### Fixes from the six-domain review

| Domain | Finding (before) | Fix |
| --- | --- | --- |
| Layout | At 390px the round-timer label (“Waiting for settlement” / “Expected settlement”) was squeezed to 65–82px. It wrapped into two lines beside “Unknown” or the countdown (seen on the probe's 390 screenshot and `live-mocked-390.png`) | `src/style.css:2527` (560px and below): the label now sits on one line above the value. `src/style.css:2559` (360px and below) lets it wrap in a balanced 96px block. Checked on the new 390 screenshots |
| Writing | The feed notice read “Wheel feed offline / unknown — Wheel feed offline. Retrying automatically. Automatic retries use a gradual backoff.” That repeats itself, and the slash heading is vague | `src/api.ts:101-102` now gives only the cause. `src/App.tsx:1040` titles the notice “Live feed unavailable”. `src/App.tsx:1047` says once that it retries (or gives the next-attempt time) |
| UI / writing | The scoreboard said “You” twice: inside the avatar and again in the caption below it | `src/App.tsx:967`: a decorative (`aria-hidden`) `UserRound` icon replaces the avatar text. `src/style.css:897-901` scopes the sparkle positioning |
| Share metadata | `og:image` was the relative `./social.png`, which many link unfurlers do not resolve | `index.html:18-30`: the image is now an absolute URL on the live domain, plus `og:url`, width/height/alt and a canonical link. A curl of the live site returned 200 `image/png` for that image URL. Page assets stay relative (`./assets/…`, `./favicon.svg`) |

Accessibility, typography and colors: no new findings. The same tokens and font are used, axe still reports zero violations in all four tested states, and the probe's contrast/type checks pass. Limitations that remain: no human screen-reader session, no RTL or real 200% zoom check, Safari/WebKit not available, and the animation-panel replay was not checked.

### Commands and actual results (final tree)

| Check | Result |
| --- | --- |
| `npm ci --cache .cache/npm --no-audit --no-fund` | Installed from the existing lockfile; `package.json` and `package-lock.json` unchanged |
| `npm run typecheck` | Passed, no diagnostics |
| `npm test` | **61 passed, 0 failed** |
| `npm run build` | Passed (Vite 6.4.1); `dist/index.html`, `assets/index-8-p8TxEn.css`, `assets/index-BOhpXZz4.js`, `assets/api-schemas-BuEuZ2Un.js` |
| `IMD_PLAYWRIGHT=… PLAYWRIGHT_BROWSERS_PATH=/opt/imd-tools/ms-playwright npm run test:browser` | All 14 Chromium checks passed: offline honesty and phone first screen, full practice game (90 points, streak 3, Hat Trick), save/collection/1080px PNG download, live lock and two-browser duel, confirmed resolution, eight-day return, stale and unknown-schema handling, 320/390/1440 overflow, reduced motion/pause/focus, no uncaught JS errors. Axe: zero violations in four states. WebKit unavailable |
| Pinned taste probe (`--out test/scratch/taste`) | **16/16 passed**, including layout, targets, contrast, type sizes, slogans, copy budget and share. First screen at 390: the product, the Nap/Sprint choice and the main action (lock, or “Waiting for wheel feed” plus Play practice when offline) are visible on the screenshot |
| Live public API through the app's own `fetchStatus`/`fetchHistory`/`fetchEvents`/`feedHealth`/`roundTiming`/`predict` (scratch script, Node) | Schema versions still 1/3/4 and accepted by the bundled schemas. Results: health `live`, open round 617, 24 + 28 buckets, 428 settlement events with a complete scan, a verified 600-second close window, Professor call NAP (31.7%). This is a snapshot from 2026-11-08 ~02:36 UTC, not a constant |

Bundle: **47 project files, 1,479,171 bytes**, of which `dist/` is **9 files / 658,912 bytes**, well under the 8 MiB cap. `git check-ignore -v dist/index.html` prints nothing. `.gitignore` is unchanged (55 bytes): no node_modules, cache, scratch output, source map or archive is included.

The earlier validation record follows, kept as history.


Checked on 2026-11-08 against the built static export. These are worker-run checks, not an independent certification or Genesis approval. No deployment or public repository URL is claimed before the IMD publishing handoff.

## Revision: pending calls older than seven days

Finding `dfbadd458610d9c97744e87c3c8668a30f6a297160e1694c9d42aa2145caa9b3` reproduced before the fix. The new test in `src/events.test.ts` supplied schema-valid safe settlements for #101 on November 8 and #1253 on November 16. The original loader returned only `[1253]` with `complete=true`, and its assertion failed because #101 was absent. These dates and outcomes are synthetic review fixtures, not real-world observations.

Fixed at `src/api.ts:195`: pending live calls from the matching installation extend evidence retention through their oldest lock time. Cached overlap cannot end a scan while an older settled call still lacks evidence. Forty pages is now a per-poll budget with a continuation, rather than a permanent scan limit. Page checkpoints survive transient failures; documented HTTP 400/409 cursor invalidation starts a fresh scan. `src/useFeed.ts:27` receives saved entries from `src/App.tsx:94` through a ref, preserving the existing polling loops. Scoring order and the seven-day AI filter in `src/game.ts` are preserved.

Actual revision checks:

| Check | Result |
| --- | --- |
| `npm ci --cache .cache/npm --no-audit --no-fund` | Installed the existing lockfile; root manifest/lockfile unchanged |
| `npm run typecheck` | Passed, no diagnostics |
| `npm test` | **61 passed, 0 failed**, including 8 new evidence/backfill regressions |
| `npm run build` | Passed; final export rebuilt after the last source change |
| `npm run test:browser` with the supplied Playwright/Chromium paths | All Chromium flows passed, including the return scenario below; no uncaught JavaScript errors |
| Pinned taste@1 command below | **16/16 passed**; no overflow, overlap or clipped text at 320/390/1440 |
| Axe audits | Zero violations: 44 checks each on live/practice boards; 19 each on practice/recovered result dialogs |
| WebKit | Runtime unavailable; Safari remains unverified |

The browser regression in `test/browser.mjs:410` locks #101, reloads eight virtual days later and locks #1253 while both remain unresolved. A first page supplies #1253, cached recent rows and a record past the training cutoff; the needed #101 is behind an older cursor. An injected 409 error keeps both calls waiting and shows the feed error. After the virtual backoff, the restarted scan retrieves the old evidence, then resolves #101 for **30** and #1253 for **35** points, with streak **2**, **two live artworks**, **35 weekly points** and **65 all-time points**. Reloading does not duplicate points or reveal an old result again. The unit suite separately verifies 41-page resumption and transient network failure checkpoints.

Screenshots inspected: `test/scratch/browser/live-mocked-390.png`, `return-pending-390.png`, `return-result-390.png`, and `test/scratch/taste/shots/390.png`. The primary controls fit the 390×844 screen; the recovered result shows +35 with +10 call, +15 bracket and +10 streak, without clipped text. All browser feed fixtures are isolated from production.

The pinned Better Interface guide was applied to the changed states across all six domains: accessibility (native controls/dialog, visible error, keyboard/reduced-motion checks and zero axe violations); layout (390px result/first-screen inspection, automated 320/390/1440 checks); writing (existing explicit waiting/error copy, no fabricated result); typography (existing Outfit scale/weights, probe passed); colors (existing rendered pairs pass the probe); UI (pending, cursor error, recovered result, collection, score switching and reload). No visual source change was required. Existing limitations below still apply, including human screen-reader use, RTL/200% zoom, Safari and animation-panel replay. Live API retention and availability cannot be guaranteed; genuinely absent evidence remains waiting. The revision uses only the documented event endpoint/cursor, rechecked against the [official API reference](https://docs.squeekchain.com/developers/api/) on 2026-11-08; actual public gameplay was not observed anew in this revision.

Revision integrity audit: **47 eligible files, about 1.41 MiB**, including **9 files / 657,909 bytes** in `dist/`, below the 8 MiB limit. Local HTML/CSS/module asset references resolve; the export is not ignored. The root package manifest, lockfile and ignore file are unchanged. No symlink, dependency/cache directory, source map or package archive is included. `.imd-responses.json` contains the exact finding ID and `fixed` verdict; the worker consumes that response separately.

Complete for this revision’s scope. The following original-build record is preserved as historical evidence; its direct public-feed observations were not re-run for the revision.

## Results

| Check | Actual result |
| --- | --- |
| Production build: `npm run build` | Passed; Vite 6.4.1, relative static assets, no chunk-size warning after splitting API schemas |
| TypeScript: `npm run typecheck` | Passed, no diagnostics |
| Unit tests: `npm test` | 53 passed, 0 failed; Node test runner, no network required |
| Interaction suite: `PLAYWRIGHT_BROWSERS_PATH=/opt/imd-tools/ms-playwright npm run test:browser` | Chromium passed; virtual clock and clearly mocked, schema-valid feed fixtures |
| Axe: practice result dialog | 19 checks passed, zero violations |
| Axe: live board and practice board | 44 checks passed in each state, zero violations |
| Pinned taste@1 probe | 16/16 checks passed on final export |
| Responsive views | Play, Collection and Achievements had no horizontal overflow at 320, 390 and 1440px |
| First screen | Both live and practice primary controls measured within 390 × 844; screenshots inspected |
| Public source schemas | Real status v1, settlements v3 and events v4 responses passed the downloaded official schemas |
| Actual feed inspection | Complete seven-day public event scan; real data displayed in Chromium through unchanged-response transport forwarding; no uncaught JS errors |
| WebKit | Runtime unavailable; Safari behaviour not verified |
| Hosting/public GitHub | After-review IMD handoff; not executed or invented by this worker |

The exact final probe command was:

```sh
IMD_PLAYWRIGHT=/opt/imd-tools/playwright-mcp/node_modules/playwright/index.mjs PLAYWRIGHT_BROWSERS_PATH=/opt/imd-tools/ms-playwright node .imd/reads/skills/design-critic/scripts/taste.mjs --out test/scratch/taste
```

Probe categories passed: render, layout, targets, contrast, casing, tracking, eyebrows, type sizes, typefaces, glyphs, slogans, repetition, copy budget, chrome, motion and share. It measured no horizontal scrolling, text overlap, floating-control obstruction or clipped text at 320/390/1440. On the tested initial phone screen it counted 16 words of prose, below its 60-word bar. It found one font family and three weights. Its minimum 24px/spacing target check passed; many secondary controls are smaller than the preferred 44px comfort size. Main lock/reveal actions are at least 46px tall.

## Meaningful game checks

The unit suite covers the actual source functions, including:

- AI formula, deterministic replay with identical/reordered history, seven-day and future-time filtering, empty-hour/history priors, bracket tie-breaking, and honest treatment of zero-work and mixed positive batches.
- Both sides’ identical scoring; exact bracket edges; the first +5 and growing streak bonus; reset on a wrong call; positive work with zero completed blocks.
- Confirmed NAP, small Sprint and large Sprint resolution; range coverage; future/pre-lock rejection; waiting with no settlement; idempotence; ordered streak resolution; live/practice separation; fair friend-duel streaks.
- Open-round plus-one arithmetic, unavailable anchors and safe-integer limits; exact closing windows; closed/oracle/buffer guards; settlement spacing bounds.
- Hardware freshness, absent/offline/stale/simulated/replay/fault/future data; malformed and unknown-version responses; wrong-network and stale history rejection; Retry-After seconds/date parsing and cooldown.
- Weekly UTC boundaries, settlement-time scoring, achievement separation and week completion, collection thresholds, UTF-8 duel encoding and malformed links, stable artwork RNG, fixed practice outcomes and counter/day-reset handling.

The browser suite performed the complete primary flows on the built export:

1. Offline live game: no fake hamster or sleeping state, unknown metrics, blocked live lock, Practice accessible on the first phone screen.
2. Three practice rounds: Nap, six-block Sprint, 78-block Sprint. The player earned **90 points**, reached streak **3**, collected **3 artworks** and unlocked Hat Trick. Live score/history remained empty.
3. Reload persistence, collection navigation, native art modal/Escape, a verified **1080px PNG** download, nine achievement cards and the required About attribution.
4. Schema-valid mocked live feed: lock a call, generate its duel link, open it in a second independent browser context, and make a same-round friend call. No reveal occurred before a settlement was supplied.
5. Advance the virtual clock and supply a covering confirmed settlement: reveal, **player +30 vs friend 0**, Duelist unlock and no duplicated points after reload.
6. Stale and unknown-schema paths, visible explanations, responsive view switching, pause/resume control, reduced-motion loop suppression and visible keyboard focus. No uncaught JavaScript errors were recorded across these flows.

Mocked tests never enter the production build. `test/fixtures/README.md` identifies their source and in-memory rewrites; the test script intercepts requests only in its own browser contexts. Test clocks are advanced programmatically, with no real ten-minute wait.

## Real endpoint evidence and limits

An additional direct check used the production `fetchStatus`, `fetchHistory`, `fetchEvents`, `feedHealth`, `roundTiming` and `predict` functions against the actual public HTTPS endpoints. It completed successfully with **24 day buckets, 28 week buckets, 420 settlement events and 350 zero-work events**, a complete scan, known closing schedule and LIVE hardware health at that observation. These are historical validation counts, not constants in the site.

A separate Chromium inspection displayed real **open round #609** and a **01:16:21 UTC** LIVE reading on 2026-11-08. Its Professor inputs showed 14% sprint for that UTC hour, last three known outcomes all Sprint, and a blended 40% Sprint probability, yielding Nap. The browser test runner forwarded the public HTTPS response bodies **unchanged**, without replacing dates, work, hashes or installation IDs. This transport bridge was needed because direct browser external-network access was unavailable in this check environment. It is a scratch-only check, not a deployed proxy or backend. No uncaught JavaScript error was recorded.

Observed real-world live results are not claimed for the mocked duel test. API uptime, actual long-term accuracy, multi-day user retention, and a real friend opening a published URL remain outside this local validation. The app’s browser network access after publishing depends on the documented public CORS service.

## Better Interface review across all six domains

| Domain | Coverage and fixes | Remaining limits |
| --- | --- | --- |
| Accessibility | Native buttons/disclosures/dialogs, explicit names and pressed states, labels, skip link, 3px focus ring, keyboard/Escape, reduced motion, manual pause, polite status/toasts; axe and probe checks passed | No human screen-reader or switch-device session; real iPhone/Safari unavailable |
| Layout | Real export inspected at 320/390/1440; main controls moved up on phones; 24-hour targets changed to three rows of eight; open galleries and shared alignment; no measured overflow/overlap/clipping | RTL, OS text scaling and actual browser 200% zoom not verified |
| Writing | Plain adjacent directions, explicit unknown/waiting/practice/local labels, no accuracy or financial promise; removed redundant two-part slogans; required community attribution present in About/README | About intentionally contains detailed rules below the game; does not fit in one screen |
| Typography | Local Outfit WOFF2, three used weights, six semantic size roles, tabular metrics, readable input sizes, wrapping full hashes; probe counted five repeated sizes at both sampled widths | Platform rasterisation and font fallback differences not exhaustively tested |
| Colors | One orange action, separate Nap/saved states, neutral paper surfaces; secondary text darkened after measured contrast failures; final probe/axe passed | Canvas artwork is decorative and has no text contrast claim; no dark mode |
| UI details | Selected/check states, locked/pending/result/error/empty flows, saved art, native modal containment/focus return, no border-plus-shadow cards, reduced-motion and pause paths | Safari rendering and animation-panel 10% replay not verified |

### Fixed findings

| Finding | Source and disposition | Evidence |
| --- | --- | --- |
| Secondary text contrast on scoreboard/activity surfaces | `src/style.css:17`, `--muted` changed from `#746f66` to `#686258` | Initial probe reported 4.16:1 and 4.3:1; corrected token pairs compute to **5.03:1** on `#eeeae1` and **5.20:1** on `#f0eee7`; final rendered checks pass |
| Narrow, crowded phone hour-strip targets | Final mobile `.hour-strip` rule in `src/style.css:2444` | Initial 24-column phone targets were 13px wide. Three rows of eight with hour labels now meet the probe’s target/spacing check |
| Main prediction pushed toward/below the phone fold | Final `.mobile-nav`, `.site-header`, `.wheel-scene` and `.game-board` mobile overrides in `src/style.css:2503`; nav in `src/App.tsx:521` | Shorter nav and scene bring both practice and live lock actions within 390 × 844; browser assertions and inspected screenshots confirm |
| Repeated two-beat slogans | `src/App.tsx:484` intro, Professor line, scoreboard empty state and gallery heading | Initial probe counted six; final probe counted two, including a data row false-positive, below its limit |
| Unnamed phone help control and dialogs | `src/App.tsx:472`, `aria-label="How to play"`, result/art `aria-labelledby`, `restoreFocus` | Explicit accessible names and native modal focus paths; tested states have zero axe violations |
| Historical pattern implied a percentage unsupported by bucket totals | `src/App.tsx:1433`, `ActivityStrip` | Now displays active-round counts and intensity relative to the busiest six-hour window; UI states the coarse resolution |
| Counter resets could misstate a since-visit distance | `src/game.ts:24`, `observeDistance`; use in `src/App.tsx` | Reset or epoch change makes the current delta unknown; both reset paths covered by tests |
| Feed installation transition could mix history with a different current status | `src/useFeed.ts:102`, `eventsInstallationId`; guards in `src/App.tsx:173` | Both the lock state and live reconciliation require matching event/status installation IDs; ordinary matching-feed flows pass |

Other measured solid token pairs: ink `#302d29` on page `#f7f6f2` **12.67:1**; action text `#fffefa` on `#be4729` **5.05:1**; focus `#246d99` against `#fffefa` **5.59:1**. These ratios were calculated using WCAG sRGB luminance for the identified source pairs; they are not claims about text placed over artwork.

## Requirement coverage

| Brief requirement | Delivery |
| --- | --- |
| Original browser game, immediately playable, no account or wallet | Arena with explicit live/practice switch; fixed, separately labelled instant practice |
| Nap/Sprint and four Sprint brackets | Interactive grouped choices, validation, immutable local lock |
| Professor with deterministic rule and visible inputs | Pure tested formula, disclosure, no accuracy claim |
| Weekly/all-time head-to-head and streak scoring | Separate local player/AI totals and result breakdown |
| Resolve only on covering settlements | Safe protocol settlement events and range/date checks; waiting state |
| Live wheel with speed, sleep and honest unknown states | Timestamped LIVE/UNKNOWN/OFFLINE strip and original pixel scene |
| Open-round distance | Scoped, measured since-visit delta; complete pre-visit distance unavailable from the supplied status API and never invented |
| 24-hour activity from seven-day buckets | Relative activity strip; actual six-hour resolution disclosed |
| Deterministic art and local collection | Transaction-hash seeds, NAP skies, work/block-driven swirls, collect and PNG download |
| Nine local achievements and toast | All nine named unlock rules present; separate practice/live storage |
| Shareable friend duel | UTF-8 URL hash, same-round/installation/deadline checks, second-context resolution; editable/local nature disclosed |
| Points only, Sepolia only, no mainnet/financial/promotional promise | Explicit UI/About/footer/README disclaimers; no wallet or write API paths |
| Required project attribution and Genesis instructions | README first paragraph, About, IMD footer/meta attribution, exact-value owner verification steps |
| Source, manifest/lock, relative static export | Root React/TypeScript source and configuration, package lock, `dist/index.html` and local assets |
| Title/favicon/description/OG metadata | Exact requested title, original SVG favicon, local 1200 × 630 PNG social image |
| Six-domain review and design documentation | This review plus final-source `DESIGN.md` |
| Hosted site and public GitHub URL | IMD post-review publication/source handoff; no fabricated destination, no local publishing command |

## Submission integrity and budget

The submission was enumerated using read-only `git ls-files --cached --others --exclude-standard`. An intermediate audit before this validation document and the final installation guard contained **45 files / 1,434,131 bytes**, including a **9-file / 656,894-byte** static export, with **zero symlinks**. The final source and this document remain well below the **8,388,608-byte** cap; the final audit is recorded below.

The ignore file is **55 bytes**, within its explicit 256-byte path budget. `git check-ignore -v dist/index.html` prints nothing: the export is eligible for the worker’s commit. Generated `node_modules` directories, task-local npm caches and scratch outputs are ignored. No package archive, vendored registry, source map, nested Git checkout or environment file is part of the deliverable. The worker, not this agent, stages and commits the final tree.

Scratch screenshots and automated reports exist only under `test/scratch/` during checking and are removed by the job pipeline. Durable evidence here is the check description and actual result, not a claim that those scratch files are submitted. The pinned `.imd/reads/` inputs are likewise not runtime or build dependencies.

Final integrity audit: **46 files / 1449637 bytes** in the submission bundle; **9 files / 657145 bytes** in `dist/`. Both are below the 8 MiB submission cap. All local HTML/CSS asset references resolved, the share PNG is 1200 × 630, test fixtures are absent from runtime bundles, and the export is not ignored.
