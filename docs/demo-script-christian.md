# Demo Script — Christian Harrison (The Alex Theatre)

> **Audience:** Christian Harrison, owner/operator. Not technical. Currently paying
> **~$365/mo** ($175 Boxoffice site+Google showtimes+email, ~$190 Square email for 6,000 contacts).
> **Purpose of this doc:** the talking points + running order for the demo, written from *his* seat.
> **Status:** draft for dry-run with Tim + Renato. Not client-ready until the blockers below are cleared.

---

## 1. The one thing to understand before writing a single slide

Christian has already sat through a website demo — Boxoffice's, over four months
(Mar–Jul 2026). He never launched it. Read his emails and the pattern is unmistakable:
**he did not ask for a prettier website. He asked to be able to run his theater, and
kept getting told "you'll have to do that manually" or "we can't."**

Direct quotes, in his words:

| What he asked | What Boxoffice told him |
|---|---|
| "I still need an easy to use system for clients to buy tickets" | Paste individual Square links into SET, one per showtime |
| Faster banner rotation (5 seconds) | "We can't update the rotation speed" |
| Two movies side by side, one row | "the widget is set to display one film at a time" |
| Delineate large vs small screen | "There is not enough delineation… in fact there is zero" |
| Featured movies on homepage | "You would just need to add/remove the movies on this page manually" → his reply: **"And I'd have to do that for each movie?"** |

**So the demo's spine is not "look what we built." It is: every place you were told
*no* or *do it by hand*, this does it for you — and the thing they couldn't do at all,
take money for tickets, is already working.**

Corollary: **do not open with the admin panel.** That's our pride, not his problem.
Open where his customer opens: a phone.

---

## 2. Blockers — fix before the call, or the demo backfires

| # | Blocker | Why it kills the demo | Owner |
|---|---|---|---|
| B1 | **Capacity doesn't follow the screen.** It *is* operator-editable, but every entry point pre-fills **50** regardless of screen — `admin/showtime-edit.php:15`, `showtime-scheduler.php:119`, and the POST fallbacks (`?? 50`), matching the schema default (`database/schema.sql:49`). His screens are **480** and **15**. | In Act 4 he picks "small screen" and the form still suggests **50 seats for a 15-seat room**, and accepts it silently. That's the exact "you didn't listen" moment to avoid. Fix = default capacity from the selected screen. | eng |
| B2 | **Demo data is generic.** Seed data is Mandalorian/Mario Galaxy. Boxoffice already demoed him "The Mario Galaxy Movie." | Fake movies = another vendor demo. **Load this weekend's actual Alex schedule**, including his own example shape: small screen 4:30pm + 7:30pm, two different films. | eng, needs his schedule |
| B3 | **Google showtimes — open question, not a settled finding.** Boxoffice's SET feeds his showtimes to Google, and in the thread that's a *separate product* ("Referral Ticketing," run by Júlio César Pereira, with Google completing setup on their end after receiving a logo). Our JSON-LD is **probably not** equivalent to being a Google showtimes data partner — **confirm before the call, don't assert either way.** | His highest-value Boxoffice feature, and he will ask. Without an answer the pitch reads as "lose Google to gain checkout." | **Tim — verify, then decide the position** |
| B4 | **No newsletter/email marketing exists.** Verified: zero subscriber/newsletter code in the repo. He asked Boxoffice to automate his newsletter and import his 6,000-person list. | ~$190/mo of his spend is email. If we imply we replace Boxoffice wholesale, we get caught. | Tim — scope call: build it, or explicitly leave it |
| B5 | **Kiosk not live-verified.** `public/kiosk/index.php` exists (327 lines) but was unbuilt at the last audit and I have no record of an end-to-end run. | Never demo an unrehearsed screen. | eng — verify or cut from demo |
| B6 | Oversell window: no capacity **hold** during the payment window. | Only matters on the 15-seat screen, where it matters most. Disclose, don't hide. | disclose in Act 6 |
| B7 | **The two most quotable "they said no" items aren't built yet.** Homepage has no Large/Small **sections** — only a small badge on each poster in one flat row (`public/index.php:74,80`). Hero rotates **building photos every 4.5s**, not movies (`main.js:80`, `main.css:1329`). | Act 1 opens on "large and small screen visibly separated." A badge in a single row is arguably the *same* complaint he made to Boxoffice — *"there is zero delineation."* And he asked them for 5-second rotation specifically. Demoing 4.5s photo rotation instead of films spends our best moment on nothing. Also `tickets.php:52` still reads *"Call ahead to reserve seats, **especially for the small screen**"* — the live site contradicts Act 1's "must be purchased online" one click from the nav CTA. **Plan written; three decisions still open with Tim. Must ship before dry run #1.** | eng |
| B8 | **Act 4 breaks naive Large/Small sectioning.** Creating a movie pre-selects screen **"Both"** (`movie-edit.php:13,566`) — which is what *enables* the per-showtime screen picker (`:644`). So a film Christian creates in Act 4 with only small-screen showtimes saves as `movies.screen='both'` and, if the homepage sections by movie-level screen, appears under **"ON THE LARGE SCREEN"** too. | This lands in the one Act where he is holding the phone, and it's the same "you didn't listen" failure as B1. Fix: derive section membership from the film's **active showtime screens** (already present on every row via `MovieRepo::attachShowtimes()` — no new query), falling back to `movies.screen` only when a film has no showtimes. | eng |

**Already in our favor, no work needed:** he asked Boxoffice for *"two movies side by side, one row"* and was told the widget shows one film at a time. Our poster row already does this — **show it, don't explain it.** Put two films on screen and say nothing; let him notice.

**Rule: anything not personally clicked end-to-end the morning of the demo gets cut
from the demo.** Boxoffice showed him half-finished examples for four months. We don't.

---

## 3. Running order

Total ~25 min of demo, ~15 min conversation. Do it on a **phone, screen-mirrored** —
his customers are on phones and his complaints were all about phone-sized layout.

### Act 0 — Frame it in his words (60 sec)
> "You wrote in July that you still needed an easy way for people to buy tickets.
> Everything else is secondary. So that's the first thing I'm going to show you, and
> I'm going to show it to you the way your customer sees it."

No company intro, no stack, no roadmap slide. **Do not say the word "Boxoffice" first
— let him raise it.**

### Act 1 — A customer buys a small-screen ticket (5 min) ← *the whole demo lives here*
Homepage on the phone → large and small screen visibly separated → tap a **small screen**
showtime → "must be purchased online before arrival" is right there → pick adult/child
(**$5 / $3**, his real prices) → pay by card → **QR ticket arrives.**

Say out loud: *"That's real money. It lands in your account, not ours."*

### Act 2 — Friday night at the door (3 min)
The check-in screen (`/checkin.php`): scan the QR, it turns green, and it **can't be
scanned twice**. Then: *"Your 15 seats are pre-sold and accounted for. Your 480 stays
first-come-first-serve exactly like it is today — we didn't take that away from you."*

That last sentence matters. He never asked us to change how the big screen works.

### Act 3 — The concession stand (3 min)
Staff register (`/pos`) ringing a sale, stock going down by itself, and the reorder
warning. Kiosk **only if B5 clears**.
His framing: *"You said take the prices off the website. Prices still live in one
place — here — so the website can just list what you offer."* (That was his actual
request: list, no prices.)

### Act 4 — Christian in control (5 min) ← *hand him the phone*
This is the emotional center of the demo. **Let him do it, don't show him.**

Have him add a movie and put **two showtimes on the same screen on the same day** —
his own 4:30 Hail Mary / 7:30 Hoppers example — choosing the screen on each showtime.
Then reload the public page in front of him.

> "You called same-day rotation the most complicated part of what you do. There's no
> widget to hand-edit. Nothing to update per movie. You set the showtime, the site is
> already right."

Callback to *"And I'd have to do that for each movie?"* — answer: **no.**

### Act 5 — What he made (3 min)
Reports (tonight / today / week / month, top sellers), occupancy, transactions, and a
void. Keep it to 3 minutes; he's an operator, not an analyst.

### Act 6 — Straight talk (5 min)
Say the gaps out loud, unprompted: B3 Google showtimes, B4 email/newsletter, B6 the
oversell window on the 15-seat screen, and that his domain isn't switched over yet.

**Volunteering the gaps is the differentiator.** He spent four months with a vendor
who kept saying "just about ready for launch." Being the first person to tell him what
*doesn't* work is how this stops being another demo.

Then the model, in his own framing: he proposed *"you get paid a convenience fee and
the retailer me gets the ticket fee."* Put a number on it against his current ~$365/mo.

### Close — one decision, not a menu
Ask for exactly one thing. Recommended: *"Give us this coming weekend's real schedule
and let us run your small screen — 15 seats, online only — live. Everything else stays
where it is."*

Low risk for him (15 seats, not 480), real money, and it proves the one thing he's
asked three vendors for.

---

## 4. Language rules

| Never say | Say |
|---|---|
| widget, CMS, schema, repo, deploy | your website, your movie list |
| Stripe, webhook, PaymentIntent | card payments, the receipt |
| showtimes table, ENUM | the showtime you set up |
| "we could build…" | "this does…" *(only demo what exists)* |
| SET, JSON-LD, referral ticketing | Google showtimes |

Two more: **his screens are "large screen" and "small screen"** — his words, used
consistently across every email. Not "auditorium 1 / 2" (that was Boxoffice's framing
and he never adopted it). And **do not badmouth Boxoffice or Carly.** She was
responsive and he never complained about her — he complained about what the product
couldn't do. Punching down at her makes us look small; let the product contrast speak.

---

## 5. The questions he will actually ask — and who answers them

Written as *his* questions, not our features. Rehearse these; a fumbled answer here
undoes three good Acts. Rows marked **NEEDS TIM** have no agreed answer yet — do not
improvise one on the live call.

| He asks | The answer | Status |
|---|---|---|
| *"What happens to my Google showtimes?"* | Boxoffice feeds Google through a separate product (SET / referral ticketing, Júlio's team). We have structured showtime data on the site, which is **not** the same as being a Google data partner. | **NEEDS TIM** (B3) — verify, then pick: keep Boxoffice for Google only, or pursue partner status ourselves |
| *"What about my 6,000 emails?"* | Not built. Nothing in the product sends newsletters today. | **NEEDS TIM** (B4) — build it, or say plainly "keep Square for email for now" |
| *"Who holds the money?"* | Intended answer: he does — card payments land in **his** account. This is the single strongest line in the demo, which is exactly why it can't be guessed: the live Stripe keys come from GitHub secrets into a gitignored `public/config/stripe.php`, and **nobody has confirmed whose Stripe account they belong to.** If they're Aslan's today, the line is false — said twice, to family. | **NEEDS TIM** — confirm account ownership, and the path to Christian's own account if it isn't already |
| *"What does this cost me?"* | Against his current **~$365/mo** ($175 Boxoffice + ~$190 Square email). His own proposed model: *"you get paid a convenience fee and the retailer me gets the ticket fee."* | **NEEDS TIM** — the fee number |
| *"What if the internet goes down on a Friday night?"* | Half-answer only. True and safe: the 480 large screen is unaffected — cash/card at the door exactly as today. **Do not claim a door fallback for the small screen:** I found no printable door manifest / ticket-list export anywhere in `admin/`. Honest answer is *"no offline check-in today"* → move it to Act 6's volunteered gaps rather than answering it as a feature. | Answered as a **gap**, not a capability |
| *"Do I have to do this for every movie?"* | **No.** Set the showtime and the site is already right. This is a direct callback to his own words to Boxoffice — use his phrasing. | Answered (Act 4) |
| *"Can I still change prices / take prices off the site?"* | Prices live in one place. The public page lists what he offers without prices — which is what he asked for. | Answered (Act 3) |

---

## 6. Dry-run plan (Tim + Renato + eng)

1. **Pre-work (eng):** clear B1, B2, B5. Click every Act end-to-end on a phone.
2. **Dry run #1 — Tim as Christian.** Tim plays him *skeptically*, including the
   questions he'll actually ask: "what happens to my Google showtimes," "what about my
   6,000 emails," "who holds the money," "what does this cost me," "what if the internet
   goes down on a Friday night." No slides. Time it.
3. **Renato's pass.** Renato drives Act 4 blind — no coaching. If he stumbles anywhere,
   Christian will too, and that spot is a product bug, not a demo bug.
4. **Dry run #2** with fixes, run once clean, then stop rehearsing.
5. **Live demo.** One driver + one navigator. Navigator watches Christian's face, not
   the screen, and calls the cut if an Act is landing flat.

**Open questions for Tim before dry run #1:** the B3 Google position, the B4 email
scope, and the actual fee number for Act 6. All three are business calls, not
engineering ones.

---

## 7. Working through it with you — session agenda

Tim's ask: *"let's you and I get together, and then I'll have you just work through
that demo process with me,"* with **Renato on the call too**, before we take it to
Christian. Three sessions, in this order. Don't collapse them into one.

### Session A — Tim + eng, ~45 min (no Renato yet)
Purpose: agree the **story**, before polishing the clicks.

1. Walk Acts 0–6 out loud with no screen. 10 min. If the story doesn't hold without a
   screen, the screen won't save it.
2. Tim plays Christian **skeptically** and fires §5's seven questions in any order.
3. Land the three business calls: **Google position, email scope, fee number.** These
   gate everything else — the demo cannot be rehearsed around unanswered pricing.
4. Decide what gets **cut**. A 25-minute demo with 6 solid Acts beats 40 minutes with 8.

**Tim brings:** the three answers, and Christian's **real weekend schedule** (B2) —
without it we demo fake movies to a man who's already been demoed fake movies.
**Eng brings:** B1, B5, B7 cleared, or a straight "not cleared, here's the cut."

### Session B — add Renato, ~60 min
Purpose: find the **product** bugs, not the demo bugs.

1. Renato drives **Act 4 blind** — no coaching, no hints. Where he hesitates, Christian
   will hesitate. Log every hesitation as a bug, not a rehearsal note.
2. Renato drives Act 1 on a **phone** as a first-time customer.
3. Assign the live-call roles: **one driver, one navigator.** The navigator watches
   Christian's face, not the screen, and calls the cut when an Act lands flat.
4. Renato owns the post-demo follow-up doc — whatever Christian asks for on the call
   gets written down live, and sent same-day.

### Session C — one clean run, then stop
Run it once end-to-end with fixes in. If it's clean, **stop rehearsing.** Over-polished
demos read as sales; Christian has had four months of that.

### Then: Christian
He's your brother, so the read on readiness is yours. One flag from the product side:
the honest version of this demo says *"here's what works, here's what doesn't yet."*
That plays very differently coming from family than from a vendor — it's an advantage,
but it also means a gap he discovers himself costs more than one we volunteer. Act 6
exists for that reason; don't let it get trimmed for time.

**Hard rule, restated:** anything not personally clicked end-to-end the morning of the
call gets cut.
