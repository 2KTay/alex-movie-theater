# Status check — Christian's 5 requirements, both codebases

**Read-only audit. No code changed. 2026-08-04.**

## Which repo is "Renato's codebase"

The `pages/` + `templates/` + `includes/` paths in the request match **`AslanBuilt/alex-movie`**
— the 61-commit from-scratch rebuild, branch `main`, HEAD `390a6da`. It has **no shared history
with ours** and is live at `parityrfp.com/cs/alex-movie/`. It was not on this machine, so it was
cloned to the session scratchpad (full history, 61 commits verified). It was **not** added as a
remote to the working repo — origin stays clean.

The other Renato repo, `AslanBuilt/alex-movie-theater`, is a fork of ours with **one** commit
(`fb8c3f3`) touching `.claude/**` + `CLAUDE.md` only — zero application code. Not relevant here.

**Ours** = `2KTay/alex-movie-theater` @ `59b977f`. It matters because `docs/demo-script-christian.md`
tracks blockers against *our* paths (`public/index.php:74,80`, `main.js:80`), so both are reported.

| | Ours (`59b977f`) | Renato (`390a6da`) |
|---|---|---|
| Layout | flat `public/` | `pages/`+`includes/`+`templates/`, thin `public/` |
| Working tree | clean except untracked `docs/demo-script-christian.md` | clean |

---

## R1 — Large/Small screen separation on the homepage

**Ours: NOT BUILT.** One flat poster carousel. Screen appears only as a per-poster badge:

```php
// public/index.php:73-80
$screen = (string) ($movie['screen'] ?? 'either');
$screenLabel = $screen === 'large' ? 'Large Screen' : ($screen === 'small' ? 'Small Screen' : '');
...
<?php if ($screenLabel !== ''): ?>
  <span class="screen-badge"><?= e($screenLabel) ?></span>
<?php endif; ?>
```

Two things to note: there are no *sections*, and a movie whose screen is `both`/`either` renders
**no badge at all** — so the current default (`both`, per `movie-edit.php:13`) is invisible. This is
demo-script B7/B8, unchanged.

**Renato: A PER-FILM SCREEN LABEL — structurally the same shape as ours, not sections.** The ADR
prose says "one poster case per screen," but the code disagrees. `Showtimes::tonight()` groups
**by film**, and says so:

```php
// includes/Showtimes.php:73-74
* Per FILM, and per film its own day. An earlier version forced one global
* day onto the whole board ...
$byFilm = [];
foreach ($rows as $r) { $fid = (int) $r['film_id']; ... }
```

So the board is one case per **film**, each stamped with its screen's name:

```php
// pages/public/home.php:88-91
<span class="board-screen">
  ...
  <span class="board-screen-sep">·</span>
  <?= esc((string) $b['screen']['name']) ?>
```

Deliberate, argued in `docs/decisions/0002-build-decisions.md` §3 ("The hero is the showtime
board"), derived from `Showtimes::tonight()` rather than a featured flag, with a designed
dark-house state. Screens are first-class rows (`db/seed.php:41-49`), so delineation is structural
rather than a string on a poster.

**Verdict:** neither repo has sections. Both label each film with its screen; Renato's label is a
text line from a `screens` row (always present, never blank), ours is a badge that vanishes on
`both`/`either`. Renato's is the more robust version of *the same pattern Christian already
rejected* — *"There is not enough delineation… in fact there is zero."* Treat R1 as **unbuilt in
both**, with Renato's data model being the better foundation to build sections on.

---

## R2 — Hero carousel size + 5-second rotation

**Ours: EXISTS, WRONG SPEED, WRONG CONTENT.**

```js
// public/assets/js/main.js:80
timer = setInterval(function () { goTo(current + 1); }, 4500);
```

Size, the other half of the requirement: the hero is a **fixed 640px** tall
(`main.css:1326-1332`, `.hero-cinematic { height: 640px; }`) with a crimson 3px bottom border. The
non-slideshow `.hero` (`main.css:179`) is padding-driven, `5rem 0 4rem`.

**4500ms, not 5000.** And it rotates four *building* photos, not films
(`public/index.php:16-30`: `hero-4.webp` "Classic white Chevelle outside", `hero-1` exterior,
`hero-2` neon sign, `hero-3` auditorium). Christian asked Boxoffice for 5 seconds specifically;
demoing 4.5s photo rotation spends the moment on nothing. Exactly B7.

Separately, a real poster carousel does exist (`main.js:256`, `.poster-carousel`) — that's the
*"two movies side by side, one row"* win Boxoffice refused. Already ours, no work needed.

**Renato: DOES NOT EXIST — by design.** Zero `setInterval`, zero rotation, no slideshow anywhere in
the repo. The only timer in the codebase is a check-in redirect (`ops.js:278`, 6000ms). ADR-0002 §3:

> Every reference the client sent (Regal, Cinemark) buries showtimes under a promo carousel — but
> that is a chain's problem, because a chain must ask which location first. The Alex has one location.

So the 5-second requirement is **not met, and not attempted** — the carousel was designed out. That
is a defensible product call that nonetheless directly contradicts a thing Christian asked for in
writing. **Business decision, not an engineering one.**

---

## R3 — "Events" renamed to "Rentals"

**Ours: NOT RENAMED.** Both nav and footer still say Events and point at the old page:

```php
// public/templates/header.php:94
<li><a href="events.php" class="nav-link...">Events</a></li>
// public/templates/footer.php:33
<li><a href="events.php">Events</a></li>
```

`public/events.php` (an events listing, `<h1>Events</h1>`) and `public/private-screenings.php` (the
actual rental page) **both exist** — and nav points at the wrong one. No redirect.

**Renato: SPLIT INTO TWO DESTINATIONS — not renamed.** Events content moved to a calendar page
(**What's On**, `pages/public/whats_on.php`, fed by `Programmes::events()`), and private hire became
its own page. `whats_on.php:6-18` states the reasoning: the old page was *"a directory of other
pages… Private hire is a service, not an event; it gets one line at the end and keeps its own
page."*

Critically, **Private Screenings is in `publicSecondary()`** — footer and drawer-below-a-rule, *not*
primary nav (`includes/Nav.php:29-36`). So a visitor does not see "Rentals" in the main nav at all.

```php
// includes/Nav.php:33
['id' => 'rentals', 'label' => 'Private Screenings', 'href' => '/private-screenings'],
// includes/Nav.php:90,93
'whats-on', 'events' => 'whats-on',
'private-screenings' => 'rentals',
```

Internal id is `rentals`; the visible label is "Private Screenings"; the route `events` resolves to
the `whats-on` nav id. **Admin was not renamed** — `pages/admin/programmes*.php` still says "+ Add event",
"An event needs a title.", `?tab=events`, `uploads/events`, and audit actions `event.save` /
`event.create`. Classic partial rename: public done, back office not.

---

## R4 — Small-screen "must be purchased online" note

**Ours: PARTIAL, AND CONTRADICTED ON A LIVE PAGE.**

Present on the film page:
```php
// public/movie.php:100
<p><strong>Online purchase required</strong> for this film. Tickets for the small screen must be
purchased online before your visit.</p>
```

Contradicted on the tickets page:
```php
// public/tickets.php:52
<p>Call ahead to reserve seats, especially for the small screen or for groups. We hold
reservations until 10 minutes before showtime.</p>
```

That second line is one click from the nav CTA and tells Christian's customer the opposite. Nothing
on the homepage. This is B7's third item, still live.

**Renato: BUILT, DB-DRIVEN, CONSISTENT.** The note rides on the screen, not the showtime:

```php
// pages/public/home.php:103-104
<?php if ((int) $b['screen']['online_only'] === 1): ?>
  <p class="board-note">Small screen seats are sold online only.</p>
// db/seed.php:47
'online_only' => 1, 'note' => 'Small screen seats are sold online only.',
```

Cleaner than ours: one flag, one source of truth, no page can drift out of sync. This is the better
implementation of the two.

---

## R5 — Rental page: no brown box, Google Form link

**Google Form: NEITHER repo uses one.**

- Renato: **zero** `docs.google` / `forms.gle` / Formspree references in the entire repo. Instead a
  native PHP enquiry form with CSRF (`csrf_verify`), a honeypot (`ref_code`), rate limiting
  (`RateGuard::allow(..., 5, 600)`), DB persistence (`Programmes::createEnquiry`), staff
  notification and audit logging — `pages/public/private_screenings.php:1-37`.
- Ours: **Formspree**, not Google — `public/private-screenings.php:92`,
  `action="https://formspree.io/f/xaqkjakn"`.

So if a Google Form link was the ask, it is unbuilt in both — and in Renato's case superseded by
something strictly better. Worth confirming the requirement is still wanted before building it.

**Renato's page was restructured**, with the reasoning in a comment at `:56`: what you get → what
happens → the price → the ask, because *"a number answers 'how much' for someone who has not yet
been told WHAT they are buying."* Classes are `steps` / `rates` / `spec` — no callout box element.

**"No brown box" — I cannot settle this from source.** Renato's CSS has no brown
(`tokens.css:127` `--text-faint: #8b7c72` is the closest, a text color). Ours has a
`highlight-box` at `private-screenings.php:19`, but its CSS border is `var(--crimson)`, not brown.
"Brown box" is a visual description; resolving it needs a look at the rendered page. **Unverified.**

---

## Agents A and B — were charts and slideshow ever committed?

**Ours: YES, both are in `master`.** Both branches are fully merged — `git branch --merged master`
lists them and `git log master..<branch>` is empty for each:

- `fix/reports-charts-print` → Chart.js 4.5.0 is live, CDN-pinned with SRI hashes
  (`admin/reports.php:235`, `admin/index.php:180`) plus `assets/js/admin-charts.js` with
  `chartWeek`, `chartMonth`, `chartToday`, `chartTransactions`, `chartScanRate` and a
  load-failure guard.
- `fix/kiosk-agent-b-banners` → the slideshow is live (`main.js:47-80`).

Nothing is stranded on a branch. The only uncommitted file in the repo is
`docs/demo-script-christian.md` (untracked).

**Renato: charts never existed and never will by default.** No file named `*chart*`, `*carousel*`,
`*slide*`, or `*banner*` was ever added across all 61 commits. `pages/admin/reports.php:5-9` says
*"Numbers only, printable, no charting library"*, and ADR-0002 §6 gives the reason:

> The old build used Chart.js and lost a session to print bugs. A charting library is ~70KB to draw
> four bars a manager reads once a week.

Grep hits on "Chart" in his history are that ADR and scaffold text, not code.

---

## Scorecard

| # | Requirement | Ours | Renato |
|---|---|---|---|
| R1 | Large/Small separation | ❌ badge only, invisible when `both` | ❌ per-**film** screen label, no sections |
| R2 | Hero 5s rotation + size | ⚠️ 4500ms, building photos not films; fixed 640px | ❌ no carousel at all (deliberate) |
| R3 | Events → Rentals | ❌ nav still "Events" → `events.php` | ⚠️ split into What's On + Private Screenings; the latter is **secondary nav only**; admin still "event" |
| R4 | Small-screen online note | ⚠️ on `movie.php`, contradicted by `tickets.php:52` | ✅ DB-driven, consistent |
| R5 | Rental page / Google Form | ❌ Formspree, `highlight-box` present | ⚠️ native form (better), brown box unverifiable from source |
| — | Charts + slideshow committed | ✅ both merged to master | ❌ charts refused by ADR; no slideshow |

## What this changes about the plan

1. **Nothing is duplicated work-in-flight — verified across all branches, not just two.**
   `git branch -a --no-merged master` returns only `fix/location-parking-layout` and
   `worktree-agent-a9304ecd778d2e01c`. Both touch `events.php`, so both were checked: the former
   changes 2 lines in `admin/events.php` only; the latter is a large stale kiosk branch whose
   `public/events.php` simply predates `EventRepo`. **Neither renames Events → Rentals, and neither
   touches `index.php`, `main.js`, or `header.php` in a way relevant to R1–R5.** Everything unbuilt
   is genuinely unbuilt. Renato's tree is clean with no branches but `main`.
2. **R4 has a reference implementation.** Renato's `screens.online_only` is the better design;
   porting the *shape* to ours fixes the `tickets.php:52` contradiction at the root instead of
   editing copy in two places.
3. **R2 is a decision, not a task.** Renato removed the carousel on principle. Ours has one at the
   wrong speed with the wrong content. Someone has to decide whether Christian gets the 5-second
   film rotation he asked for, or gets told why a static board is better — before either repo is
   demoed.
4. **Capacity defaults are wrong in both.** Ours pre-fills 50 (demo-script B1); Renato seeds
   **240 / 60** (`db/seed.php:42,46`). Christian's screens are **480 / 15**. Same "you didn't
   listen" failure, two codebases.
5. **Which codebase is the demo?** Every requirement above has a different status in each repo. This
   is the first question to answer; the fix list is different depending on the answer.
6. **R1 is the headline finding.** Both repos implement a per-film screen *label*. That is the same
   pattern Christian already called "zero delineation" at Boxoffice. Nobody has built sections yet,
   in either codebase — so the Act 1 opener of the demo script has no implementation behind it.

## Verified vs unverified

**Verified from source** (file:line quoted above): R1, R2, R3, R4 in both repos; the charts/slideshow
merge status; the full unmerged-branch audit; capacity seed values.

**Not verified — needs a rendered page, not a grep:** the "no brown box" half of R5. Renato's CSS has
no brown token and no callout element on the page; ours has a `highlight-box` whose border is
`var(--crimson)`. "Brown box" is a visual description and one look at
`parityrfp.com/cs/alex-movie/private-screenings` plus our `/private-screenings.php` settles it for
both. Flagging rather than guessing.
