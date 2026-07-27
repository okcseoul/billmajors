# AI collaboration handoff notes — billmajors.com

Bill works with more than one AI assistant on this project. This file exists so
whichever assistant picks up work next has the context the others left behind.
**Update it when you hand off work; don't just delete entries.** Newest
authoritative information is at the top; dated entries below are a historical
log.

---

## Current operating model — read this first

**As of 2026-07-27.** Same governance as `museumofthebiblekorea.com`; see that
repo's `AI_HANDOFF_NOTES.md` for the full statement.

- **Production repository:** `~/Documents/Naver Search/repositories/billmajors`.
  This is the only clone. There is a non-production leftover at
  `~/Desktop/ crossculture/BILL/` — plain files, not a git repo, drifted well
  behind. **Do not edit or copy from it.**
- **Roles:** Bill is the final human decision-maker. Claude directs. Claude Code
  verifies sync, edits, tests, commits, pushes, and performs Bill-authorized
  deployments. Codex independently reviews completed commits and gives clearly
  labeled advisory feedback.
- **Sync rule:** Claude cannot authenticate to GitHub. Claude Code performs the
  authoritative fetch. A failed fetch means "sync unknown" — never "assume
  synced," never "assume stale."

---

## Deploying — the thing most likely to catch you out

**This repo is Git-connected Cloudflare Pages. Pushing to `main` publishes.**
Push and deploy are *not* separable actions here. This differs from
`museumofthebiblekorea.com`, whose `mbk` Worker requires a manual
`wrangler deploy` and does **not** auto-deploy on push. Do not carry assumptions
from that repo to this one.

Practical consequence: "commit and push, then Bill authorizes the deploy" is not
a valid sequence on billmajors. Commit locally, test, get authorization, *then*
push.

**Edge-cache lag after deploy.** Observed 2026-07-27: for roughly a minute after
a successful Pages build, the plain production URL can still serve the previous
copy even with `cf-cache-status: DYNAMIC`, while a cache-busted request already
shows the new content. Do not diagnose this as a failed deploy. Wait, or bust
the cache, before concluding anything.

---

## Known trap: two different bilingual systems, and one silent failure mode

The site uses **two incompatible EN/KO toggle mechanisms**. Mixing them produces
content that is invisible in *both* languages, with no error anywhere.

| Mechanism | Files | Correct markup |
|---|---|---|
| **JS attribute swap** | `index.html`, `about.html` | Both attributes on one element: `<a data-en="About" data-ko="소개">About</a>` |
| **CSS span toggle** | `halfapple.html`, `decades.html` | Two sibling spans: `<a><span data-en="About">About</span><span data-ko="소개">소개</span></a>` |

The CSS files contain `[data-ko] { display: none; }` and
`body.ko [data-en] { display: none; }`. **An element carrying both attributes in
a CSS-toggle page matches a hiding rule in every language state and never
appears.**

This is not hypothetical. `halfapple.html`'s two nav links were written in the
attribute style inside a CSS-toggle page and were invisible in both languages
from creation until 2026-07-27. The page shipped with no usable navigation and
nobody noticed, because the HTML looks correct and validates fine.

**Before adding any bilingual markup, check which mechanism the file uses.**

---

## `index.html` — do not overwrite wholesale

`index.html` carries SEO and structured data not present in any working copy:
`naver-site-verification`, Open Graph tags, `canonical`, and a JSON-LD `Person`
schema. On 2026-07-27 a whole-file copy from a stale working folder was proposed
and would have destroyed all of it; Claude Code caught it by checking the diff
and hand-applied the changes instead. **Apply targeted edits to this file. Never
copy over it.**

---

## Site structure as of 2026-07-27

| Page | Nav | Toggle mechanism |
|---|---|---|
| `index.html` | Full, `id="hamburger"` / `toggleMenu()` | JS attribute swap |
| `about.html` | Present, same older pattern | JS attribute swap |
| `decades.html` | Full 6-item + accessible mobile menu | CSS span toggle |
| `halfapple.html` | Full 6-item + accessible mobile menu | CSS span toggle |
| `namecard.html` | None | — |
| `book-photos.html` | None | — |
| `halfapple-thankyou.html` | None | — |

`.assetsignore` covers `.git/`, `.gitignore`, `.assetsignore`, `.DS_Store`.
Adequate for this repo's contents — there is no private research directory here,
unlike the museum repo.

---

## Open scope — not yet actioned

1. **Nav consolidation.** Four states above. Also: "The Book" resolves to
   `index.html#book` from most pages but to `halfapple.html` from `halfapple.html`
   itself. Deliberately deferred as a separate, scoped task rather than mixed
   into content work.
2. **`README.md` says "Missionary in Seoul, Korea since June 2, 1982."** The
   date is wrong — he landed at **Gimpo on 1 June 1982**, reaching **Gwangju on
   2 June**. "Seoul" is correct: Gimpo Airport is in Gangseo-gu, Seoul.
   Corrected on `decades.html` and `index.html`; the README still carries the
   June 2 date.
3. **`halfapple.html` purchase CTA below the fold at 768px.** Top of
   `.cta-group` sits at ~1164px. Verified pre-existing (≈1163px before the
   2026-07-27 nav work), caused by `.hero-right` preceding `.hero-left` under
   900px. Not a regression; needs its own task.
4. **Single 900px breakpoint** on `decades.html` and `halfapple.html`. 1024px and
   1100px both get the desktop layout. Tested and clean, but there is no tablet
   intermediate.

---

## 2026-07-27 — Claude → whoever is next

Built `decades.html` ("The Same Years"), a bilingual two-column timeline running
1960 to the present: Korean national history in the left column, Bill's life in
the right. Source material was 84 photographs from the National Museum of Korean
Contemporary History plus *The Half Apple*. Research files for that work live in
the museum repo's sibling folder at `Cowork/ivm_project/72` and `/73`.

Also this session:

- Trimmed `index.html`'s `#story` from ten items to five, removed three YouTube
  iframes from it (they now live only on `decades.html`), corrected the arrival
  date from June 2 to June 1, and dropped out-of-order entries.
- Fixed the invisible `halfapple.html` nav described above.
- Gave both `decades.html` and `halfapple.html` the full six-item nav with an
  accessible mobile menu: `role="dialog"`, `aria-modal="true"`, a Tab focus trap
  cycling in both directions, Escape and backdrop close, focus restoration to the
  hamburger, and `aria-current="page"`.
- **On the focus trap:** it was first built as a disclosure without `aria-modal`,
  on the reasoning that a slide-out nav is not a dialog. Testing showed keyboard
  users tabbing into page content hidden behind the backdrop while mouse users
  could not reach it. The component behaves modally, so it was made modal. Do not
  revert to the disclosure pattern without solving that asymmetry.

**YouTube embeds on `decades.html`.** Three videos: ordination, wedding, Houston
sermon. They throw **Error 153** when the page is opened over `file://` — no
origin for YouTube to validate. This is not a markup fault. Test over
`http://localhost` or the live URL.
