# Final product name

Research for [issues/05-final-product-name.md](../issues/05-final-product-name.md). Ten fresh
candidates, generated and vetted. The eleven names already in the ticket's table were not re-vetted.

## What the prior failures implied for generation

The ticket's table says short evocative Latinate coinages die on category collision. That
reproduced immediately: the first three names I tried in that style — `velmo`, `talvo`, `nimbo` —
were all already taken, two of them by habits/affirmations apps. Startup-name-generator shapes
(CVCCo, four to five letters, Italianate) are exhausted in this category.

So the pool was rebuilt from **obscure concrete trade nouns** — joinery, roofing, horology, rope
work, weaving — on the theory that this is the only remaining source of words that are short,
sayable in both languages, and that nobody has reached for. This is the `stint` profile the ticket
called out: English-specific, semantically precise, unpoetic.

## Ranked shortlist

Filters applied in the ticket's order: (1) category collision on App Store / Google Play / RuStore
/ GitHub, (2) Russian readability, then say-it-out-loud, common-noun status, currency/city/brand,
and inflected forms. Trademark and domains are reported but did not drive the ranking.

| # | Name | Stores (AS / GP / RuStore) | GitHub | Russian | USPTO Cl. 9/42 | Verdict |
|---|---|---|---|---|---|---|
| 1 | **Muntin** | 0 / 0 / 0 | nothing relevant | мунтин, no shadow | 0 marks, any class | **Recommended.** Only occupant anywhere is one small unregistered software brand |
| 2 | **Purlin** | 0 / 0 / 0 | max 12★, unrelated | пурлин, clean | **live Cl. 42 reg. for AI software** | Best word in the set; the name is already someone's software brand |
| 3 | **Detent** | 3 apps / 2 apps / 0 | max 20★, unrelated | детент, near `детант` | clear (only Cl. 9 mark is dead) | Best metaphor; "detention" swamps EN search |
| 4 | **Sennit** | 0 / 0 / 0 | max 3★ | `Зенит` shadow; `сеннит` is an archaic RU noun | 0 marks, any class | Cleanest register, worst Russian |
| 5 | **Trunnion** | 0 / 0 / 0 | max 13★, unrelated | траннион, clunky | clear on register | Two live software brands trading on the name |
| 6 | **Ricasso** | 1 / 1 / 0 | all 0★ | рикассо, clean | **pending Cl. 9 + 42 for a mobile app** | Drop — a shipping app owns it |
| 7 | **Heddle** | 1 / – / 0 | 25★ + 9★ dev tools | хеддл, hard cluster | not checked | Dev-tool namespace occupied |
| 8 | **Orbin** | 0 / 0 / 0 | 127★ near-miss | орбин, reads as a surname | not checked | Pure coinage, but bland |
| 9 | **Newel** | 1 (Business) / – / 0 | **1468★ Newelle** | ньюэл, awkward | not checked | Newell Brands + Newelle |
| 10 | **Parrel** | 0 / 0 / 0 | **6090★ ParrelSync** | паррел, clean | not checked | Dead — homophone of "peril" |

---

## Method

Store checks were done against machine-readable endpoints, not by eyeballing search pages:

- **App Store** — Apple's iTunes Search API, both `country=us` and `country=ru`, `entity=software`,
  filtered to results whose `trackName` actually contains the candidate:
  `https://itunes.apple.com/search?term=NAME&entity=software&limit=50&country=ru`
  ([Apple Search API docs](https://performance-partners.apple.com/search-api))
- **RuStore** — the backend behind the catalog search UI, queried in both Latin and Cyrillic:
  `https://backapi.rustore.ru/applicationData/apps?query=NAME&pageSize=20&pageNumber=0`.
  It matches fuzzily (a query for `muntin` returns Mountain games), so a zero *exact-name* hit is
  what is being reported.
- **Google Play** — `https://play.google.com/store/search?q=NAME&c=apps`, scraped and filtered to
  title matches, then each hit's listing fetched for its category.
- **GitHub** — `https://api.github.com/search/repositories?q=NAME+in:name&sort=stars`, plus
  `NAME pomodoro habit`. Every candidate returned **zero** results for the pomodoro/habit query.
- **Language** — every candidate was checked on
  [en.wiktionary.org](https://en.wiktionary.org) for *all* language sections (this is what catches
  the `sutra` failure class), and its Cyrillic transliterations checked on
  [ru.wiktionary.org](https://ru.wiktionary.org). Transliterations were also machine-scanned for
  ~90 problematic Russian substrings (`урн`, `кал`, `рак`, `вонь`, `манд`, …) — the `diurna` check.

Per the ticket, `whois` and `dig` were not used. Domains were HTTP-fetched. A control fetch of a
nonsense domain returned no server, so the environment is not wildcarding HTTP — but **"no server
responding" still does not prove a domain is unregistered.**

---

## 1. Muntin — recommended

*The vertical or horizontal bar that divides a window into separate panes.*
[Wiktionary](https://en.wiktionary.org/wiki/muntin) · IPA `/ˈmʌntɪn/` ·
[Wikipedia](https://en.wikipedia.org/wiki/Muntin)

**Category collision — none found anywhere.**

- App Store, `country=us` and `country=ru`: **0** results with "muntin" in the track name.
  `https://itunes.apple.com/search?term=muntin&entity=software&limit=50&country=ru`
- Google Play: **0** title matches.
- RuStore: **0** exact-name hits for `muntin` and for `мунтин` (20 fuzzy results, all "Mountain"
  driving games).
- GitHub: 45 repos match `muntin in:name`, all of them Muntinlupa-City student projects (max 4★).
  `muntin pomodoro habit` → **0**.

**Russian.** `мунтин`. No entry in [ru.wiktionary](https://ru.wiktionary.org/wiki/мунтин), no
problematic substring, no near-homophone I could find. It fits Russian phonotactics and declines
regularly. This is the cleanest Russian record of any candidate.

**Say it out loud.** `MUN-tin` /ˈmʌntɪn/ vs *mountain* /ˈmaʊntɪn/ — the stressed vowels differ
clearly, so they are distinguishable spoken. Written they are one cluster apart, and search engines
will suggest "mountain". This is the name's main practical cost, and it is English-only; Russian has
no such confusion.

**Other languages.** en.wiktionary lists two sections: English and **Catalan**, where `muntin` is a
[verb form of *muntar*](https://en.wiktionary.org/wiki/muntin), "to mount/assemble". This is a
technical hit against the ticket's rule 6 — but the `ritma`/`tempa` failures were inflections *in
Russian*, the target language. A Catalan subjunctive is not something an EN or RU user meets.

**City.** `Muntin` is a substring of [Muntinlupa](https://en.wikipedia.org/wiki/Muntinlupa), a
552,000-person city in Metro Manila. It is a substring, not the name — but it is why GitHub returns
45 repos.

**Trademark.** **Zero** wordmark hits in the USPTO register containing "muntin", in any class, live
or dead. Web results that look like Justia trademark hits are goods-description matches ("muntin
bar" inside a recitation), not marks. EUIPO not checked.

**Existing software brand.** [muntin.digital](https://muntin.digital/) — "Muntin Digital —
Restaurant cost intelligence & the free operator library", Silver Spring MD, ships a paid product
called *Muntin Ledger*. Small, unregistered, different category, no app, no `.com`. This is the only
occupant of the name found anywhere.

**Domains.** `muntin.app` and `muntin.io` — no DNS. `muntin.com` — HTTP 200, a 114-byte Afternic
for-sale lander. `muntin.dev` — HTTP 525 (Cloudflare SSL handshake failed at origin), so registered
and on Cloudflare.

**Metaphor.** The bar that divides one pane of glass into discrete cells — which is what a timeline
does to a day.

---

## 2. Purlin

*A longitudinal horizontal beam bridging two or more rafters, carrying the roof along its length.*
[Wiktionary](https://en.wiktionary.org/wiki/purlin) · IPA `[ˈpɜːɹlɪn]` ·
[Wikipedia](https://en.wikipedia.org/wiki/Purlin)

**Category collision — none.**

- App Store us + ru: **0** name matches.
- Google Play: **0**. (An early scrape flagged "Purlin Co."; fetching the listing showed the app is
  actually named **Purlyu**, package `com.purlyu.app`, category Communication — not a match.)
- RuStore: **0** exact hits for `purlin` and `пурлин`.
- GitHub: 55 repos, top is
  [pku-dasys/purlin](https://github.com/pku-dasys/purlin) at 12★ (network-on-chip toolkit).
  `purlin pomodoro habit` → **0**.

**Russian.** `пурлин`. No ru.wiktionary entry, no problematic substring. Two faint textures worth
naming rather than hiding: the `пур-` onset is shared with
[пурга](https://ru.wiktionary.org/wiki/пурга) (blizzard; the idiom *нести пургу* = to talk rubbish)
and with [пурген](https://ru.wiktionary.org/wiki/пурген) (a phenolphthalein laxative). Both share
only the onset, not the word. A bilingual reader might instead render it `Пёрлин`, which surfaces
[перл](https://ru.wiktionary.org/wiki/перл) — "pearl", figuratively "a gem", ironically "a howler".
None of these is in the `урна` class; they are textures, not traps.

**Say it out loud.** Clean. The one shadow is
[Perlin noise](https://en.wikipedia.org/wiki/Perlin_noise), a near-homophone that every graphics
programmer knows. Irrelevant to users, mildly annoying to developers.

**Other languages.** en.wiktionary lists **English only**.

**Trademark — this is what costs it the top slot.** `PURLIN`, USPTO serial **97662034**,
registration **7303029**, LIVE/registered 2024-02-13, owner **Purlin Co.** (Delaware).
**International Class 042**: "Consulting, design, creation, and implementation of AI software and AI
software development tools for customers in the real estate industry."
[TSDR](https://tsdr.uspto.gov/statusview/sn97662034)

**Existing software brand.** [purlin.com](https://purlin.com/) is live: "Purlin — Real estate,
reinforced. The AI platform that powers every deal", claiming 40,000+ real estate professionals.
The company raised ~$4M and
[merged with Final Offer in February 2026](https://www.prnewswire.com/news-releases/purlin-and-final-offer-merge-to-create-real-estates-first-ai-platform-unifying-real-estate-mortgage-and-title-302676963.html).
`purlin.app` redirects to `purlin.com` — same owner. `purlin.io` is a parked lander; `purlin.dev`
has no DNS.

**Assessment.** Purlin passes both blocking filters cleanly and is the best-sounding word in the
set, with the most apt metaphor for this product — the longitudinal member the whole structure rests
on, which is exactly what the unified timeline is meant to be. Its only defect is trademark and
brand occupancy, which the ticket declares informational. But by the standard the ticket's own table
applies — `ritma` was killed by *a website*, `kairo` by *another app* — "already a live software
brand with the .com and a registered Class 42 mark" is the kind of fact this project kills names
for. Rank 2, knowingly.

---

## 3. Detent

*The catch that locks and unlocks a movement; in clockwork, the catch that releases the striking
train.* [Wiktionary](https://en.wiktionary.org/wiki/detent) · IPA `/dɪˈtɛnt/` ·
[Wikipedia](https://en.wikipedia.org/wiki/Detent)

**Category collision — nothing in Productivity, but the name is in use.**

App Store, exact name matches:

| App | Category | Bundle | Link |
|---|---|---|---|
| Detent: Strength Log | Health & Fitness | `com.ahmetenesdur.knurl` | [us](https://apps.apple.com/us/app/detent-strength-log/id6787412266) / also live in RU |
| DETENT: Cockpit Panel | Utilities | `com.sigilark.detent` | [us](https://apps.apple.com/us/app/detent-cockpit-panel/id6791303028) |
| Detent Coffee | Food & Drink | `com.illogicalproject.detent` | [us](https://apps.apple.com/us/app/detent-coffee/id6764680211) |

Google Play: *Detent: Escapement Casebook* (`com.detentescapementcasebook.nkr5218`, Tools/Family)
and *Detent: Sliding Block Puzzle* (`game.detent`, Puzzle). RuStore: **0** for `detent` and
`детент`. GitHub: `detent pomodoro habit` → **0**; nearest is
[digitaldrywood/detent](https://github.com/digitaldrywood/detent) at 15★ ("board-driven agentic work
orchestration").

**The search problem.** `detent` is a substring of *detention*. An App Store query for "detent"
returns prison, school-detention and trucking detention-pay apps in both storefronts — including
*SherHaul: Detention Pay* sitting in **Productivity** in the RU store. Nothing is named Detent
there, but the word cannot be searched cleanly in English.

**Russian.** `детент`. No ru.wiktionary entry; fits the `-ент` loanword pattern (патент, клиент,
процент) and declines naturally. One neighbour:
[детант](https://ru.wiktionary.org/wiki/детант), the political term for *détente* — one letter away,
though specialist vocabulary.

**Other languages.** English only.

**Trademark.** Classes 9 and 42 are clear. The only Class 9 mark, serial
[88259979](https://tsdr.uspto.gov/statusview/sn88259979) ("Computer mouse"), is DEAD/abandoned.
Note that **Amplifye Inc. filed four live DETENT applications in August 2026** (serials 50046596,
50066261, 50066241, 50066254) in Classes 1/5/29/32 — supplements and beverages, not software, but an
active brand build-out.

**Domains.** `detent.app` and `detent.io` are GoDaddy parking landers; `detent.com` redirects to
GoDaddy's for-sale page; `detent.dev` has no DNS.

**Assessment.** The best metaphor of any candidate — a detent is precisely the mechanism that turns
continuous motion into discrete, counted, held positions, which is what a pomodoro does to a working
day. It is defeated on the ticket's first filter: three apps already ship under the exact name, one
of them a Health & Fitness tracker that is a short drift from habits.

---

## 4. Sennit

*Braided cord made by plaiting rope yarns; also plaited straw for hats.*
[Wiktionary](https://en.wiktionary.org/wiki/sennit) ·
[Wikipedia](https://en.wikipedia.org/wiki/Sennit) ·
[Merriam-Webster](https://www.merriam-webster.com/dictionary/sennit)

**Category collision — none, and the cleanest register of the set.** App Store us + ru: **0**.
Google Play: **0** title matches. RuStore: **0** for `sennit` and `сеннит`. GitHub: 25 repos, the
largest at 3★ ([Sennit/RedisManager](https://github.com/Sennit/RedisManager)); `sennit pomodoro
habit` → **0**. USPTO: **zero** wordmark hits containing "sennit", any class, live or dead.

**Russian — this is what sinks it.** Two problems.

1. `Сеннит`, stressed on the final syllable as Russian does with `-ит` nouns (грани́т, магни́т,
   банди́т), is one voicing feature away from **Зени́т** —
   [зенит](https://ru.wiktionary.org/wiki/зенит) is both a common Russian noun ("zenith", and
   figuratively "the peak of something") and a national brand twice over:
   [FC Zenit Saint Petersburg](https://en.wikipedia.org/wiki/FC_Zenit_Saint_Petersburg) and the
   [Zenit camera](https://en.wikipedia.org/wiki/Zenit_(camera)). This is a weaker version of exactly
   what killed `kolo`.
2. `Сеннит` is itself a Russian word, if an archaic one: the Brockhaus-Efron dictionary defines it
   as a cyclic six-atom alcohol, methylinositol, found in **senna leaves** —
   [ЭСБЕ/Сеннит](https://ru.wikisource.org/wiki/ЭСБЕ/Сеннит). Senna is a laxative. Obscure, but it
   is the `pomidor` genre of problem.

**Say it out loud.** `SEN-nit` ends on *nit*, a louse egg — the `stint`/`stink` test, mildly failed.
It also has three accepted English spellings: *sennit*, *sennet*, *sinnet* (per Wiktionary), which
is bad for a name people have to type.

**Domains.** `.app`, `.io`, `.dev` all returned no DNS. `sennit.com` resolves but times out on both
HTTP and HTTPS — registered, no working server.

---

## 5. Trunnion

*A cylindrical protrusion used as a mounting or pivoting point; originally on cannons.*
[Wiktionary](https://en.wiktionary.org/wiki/trunnion) · IPA `/ˈtɹʌn.jən/` ·
[Wikipedia](https://en.wikipedia.org/wiki/Trunnion)

**Category collision — none in the stores.** App Store us + ru: **0**. Google Play: **0**. RuStore:
**0** for `trunnion` and `траннион`. GitHub: 9 repos, top
[trunnion/cargo-acap](https://github.com/trunnion/cargo-acap) at 13★; no pomodoro/habit hits.

**But two live software brands.** [trunnion.ai](https://trunnion.ai/) — "Trunnion AI · Agentic
software, accountable by design" (HTTP 200, 145 KB). [trunnion.app](https://www.trunnion.app/) —
Trunnion LLC, shipping a bicycle-service tracker called *Bike Mec* with Strava sync. Neither holds a
registered mark, but both are trading under the name in software.

**Trademark.** Classes 9 and 42 clear. Only two wordmarks: serial
[79123147](https://tsdr.uspto.gov/statusview/sn79123147) (DEAD, IC 007) and `TD TRUNNION DESIGN`
serial [99146268](https://tsdr.uspto.gov/statusview/sn99146268) (LIVE, IC 006/035, automotive).

**Russian.** `траннион` — no entry, no bad substring, but three syllables with an initial `тр-`
cluster and a `-ннион` tail. It is the hardest of the set for a Russian speaker to say and spell.
English only on Wiktionary.

---

## 6. Ricasso — drop

*The unsharpened length of blade just above the guard.*
[Wiktionary](https://en.wiktionary.org/wiki/ricasso) · IPA `/rɪˈkæsoʊ/` ·
[Wikipedia](https://en.wikipedia.org/wiki/Ricasso)

Russian is clean (`рикассо`, no entry, no bad substring) and Wiktionary lists English only. It dies
on the first filter.

- **App Store**: [Ricasso — Knife Collection Log](https://apps.apple.com/us/app/ricasso-knife-collection-log/id6774758245),
  bundle `com.ricasso.app`, Lifestyle. **Google Play**: [same app](https://play.google.com/store/apps/details?id=com.ricasso.app).
  RuStore: 0.
- **USPTO**: `RICASSO`, serial [99801812](https://tsdr.uspto.gov/statusview/sn99801812), LIVE
  intent-to-use filed 2026-05-03, owner THISISDISCO VENTURES LLC, **Classes 9, 35 and 42** — Class 9
  reads "Downloadable software in the nature of a mobile application", Class 42 is broad SaaS
  language. This is the only RICASSO mark in the register and it sits squarely in both target
  classes.
- [ricasso.app](https://ricasso.app/) is live and is that product.

Also: *Ricasso* is one consonant from *Picasso*, which is a permanent shadow.

---

## 7-10. Also vetted, ranked lower

**Heddle** — *the loom component whose eye lifts an individual warp thread*
([Wiktionary](https://en.wiktionary.org/wiki/heddle), English only, `/ˈhɛdəl/`). App Store: one
match, "Heddle" by Heddle Education Ltd (Education), live in us and ru. RuStore: 0. GitHub: the
dev-tool namespace is taken — [roackb2/heddle](https://github.com/roackb2/heddle) 25★ (terminal
coding agent), [goweft/heddle](https://github.com/goweft/heddle) 14★ (MCP policy layer),
[monotykamary/heddlework](https://github.com/monotykamary/heddlework) 9★ ("workspace for agent
sessions, task graphs"). `хеддл` has no Russian entry but the `-ддл` coda is awkward for a Russian
speaker.

**Orbin** — the only true coinage that survived: **no Wiktionary entry in any language**, and 0
exact hits on all three stores. But GitHub has
[TrianglyRU/OrbinautFramework](https://github.com/TrianglyRU/OrbinautFramework) at 127★, App Store
has near-misses ORBINE and Orbinity, and `Орбин` reads to a Russian as a surname. Bland, but if the
requirement were "means nothing anywhere", this is the one.

**Newel** — *the central post of a spiral staircase*
([Wiktionary](https://en.wiktionary.org/wiki/newel), English only). Stores are near-clean (one
"NEWEL AUCTIONS", Business). Killed by adjacency: [qwersyk/Newelle](https://github.com/qwersyk/Newelle)
at 1468★ is a well-known AI assistant, and [Newell Brands](https://en.wikipedia.org/wiki/Newell_Brands)
is a multi-billion-dollar consumer conglomerate. `ньюэл` is also awkward in Cyrillic.

**Parrel** — *the ring holding a yard to a mast*
([Wiktionary](https://en.wiktionary.org/wiki/parrel), English only). Stores clean, Russian clean.
Dead anyway: [VeriorPies/ParrelSync](https://github.com/VeriorPies/ParrelSync) has **6090★** and is
standard equipment in Unity development, and spoken, *parrel* lands within an accent of *peril* —
the `stint`/`stink` failure.

---

## Rejected during generation

Recorded so nobody re-vets them. Each has a cited reason.

| Name | Killed by |
|---|---|
| `velmo` | [Velmo Black](https://play.google.com/store/apps/details?id=com.velmoblack.app) on Google Play is a gamified 21-day habit app. Also one letter from Venmo |
| `talvo` | [Talvo affirmations](https://play.google.com/store/apps/details?id=com.app.talvo) on Google Play, plus [talvo.eu](https://www.talvo.eu/) (budgeting) and [talvo.dev](https://talvo.dev/) (dictation) |
| `kerf` | [Kerf: Calm Routine Planner](https://apps.apple.com/us/app/kerf-calm-routine-planner/id6774752692), Productivity + Lifestyle, released **2026-09-07** — "a calm home for your daily routines… build habits at your own pace". Head-on collision, four days old |
| `nimbo` | Six name-matches in the US store, eight in RU, including [Nimbo Clip](https://apps.apple.com/ru/app/nimbo-clip/id6809046688) in **Productivity**. Also `нимб` is a Russian noun |
| `tenon` | [Tenon](https://apps.apple.com/us/app/tenon/id6767007104) ships in **Business + Productivity** (construction PM, tasks and scheduling); seven Google Play apps; [Tenon Software Inc.](https://tenonhq.com/) is a funded SaaS. Also a common French noun |
| `deckle` | German verb form of *deckeln* ([Wiktionary](https://en.wiktionary.org/wiki/deckle)) — rule 6, in a major language. Plus [deckle.app](https://deckle.app) live and a "Deckle App Team" task-sharing developer on Play |
| `foliot` | [GNU Foliot](https://www.gnu.org/software/foliot/) is "a small and easy to use time keeping application" (v0.9.8, June 2018). Zero store presence, but a GNU-branded name in this exact category |
| `reglet` | [реглет](https://ru.wiktionary.org/wiki/реглет) is a Russian masculine noun (2nd declension) — rule 3, in the target language |
| `astragal` | [астрагал](https://ru.wiktionary.org/wiki/астрагал) is a Russian noun with three senses (plant genus, moulding, ankle bone) — rule 3 |
| `croze` | Ambiguous English pronunciation (`/kɹoʊz/` per [Wiktionary](https://en.wiktionary.org/wiki/croze), but reads as "cro-zee"), and a homophone of Serbo-Croatian [кроз](https://ru.wiktionary.org/wiki/кроз), the preposition "through" — the `sutra` failure class |
| `skep` | Afrikaans verb "to create" ([Wiktionary](https://en.wiktionary.org/wiki/skep)); also an Indonesian noun ("valve") — rule 3 |
| `bodkin` | [Bodkin](https://www.netflix.com/title/81423482), a 2024 Netflix series, owns the word in search. Also a moderately known English noun |
| `corbel` | [Corbel](https://en.wikipedia.org/wiki/Corbel_(typeface)) is a Microsoft ClearType typeface shipped with Windows. Stores are clean, but naming software after a system font is a permanent search collision |
| `burin` | [Town of Burin](https://apps.apple.com/us/app/town-of-burin/id6778342793) ships on the App Store — [Burin](https://en.wikipedia.org/wiki/Burin,_Newfoundland_and_Labrador) is a town in Newfoundland; rule 4 |
| `mandrel` | Transliterates to `мандрель`, which contains `манд-`. Exactly the `diurna`/`урна` pattern, caught by the substring scan |
| `quoin` | Pronounced identically to *coin* — rule 4 by sound |
| `pawl` | Transliterates to `пол`, which is Russian for floor / half- / sex — rule 3 |
| `collet` | `коллет` is one letter from `колет`, the 3rd-person singular of *колоть* — rule 6 |
| `fillister` | `филлистер` collides with `филистер`, the Russian for philistine |
| `soffit` | `софит` is an existing Russian noun (a stage floodlight) |

---

## Recommendation

**Muntin.**

It is the only candidate with a genuinely empty record on every axis the ticket says matters:

- **Category collision (filter 1)** — zero exact-name apps on the App Store in both the US and RU
  storefronts, zero on Google Play, zero on RuStore in Latin and Cyrillic, and zero GitHub projects
  in the pomodoro/habit/task space. Four of the prior eleven died here; Muntin does not.
- **Russian readability (filter 2)** — `мунтин` is not a word, contains no problematic substring,
  has no near-homophone, no brand shadow, and declines regularly. This is the cleanest Russian
  record of anything I vetted, better than `zarya`'s.
- **Say it out loud** — no ugly landing.
- **Common noun** — obscure enough that most native English speakers do not know it, which is the
  `stint` profile the ticket said worked.
- **Currency / city / brand** — none. Zero USPTO wordmarks in any class, live or dead.

What it costs you, stated plainly: English speakers will mistype it as *mountain* and search engines
will suggest that correction; it is a Catalan verb form (rule 6, but in a language neither of your
two audiences reads); and one small unregistered restaurant-software brand,
[Muntin Digital](https://muntin.digital/), already uses the word. None of these is what the ticket's
history shows kills names.

**If you want the better-sounding word, take Purlin and accept the trade knowingly.** Purlin has an
identically perfect store record, cleaner English phonetics, and the better metaphor for this
product — the longitudinal member the whole roof rests on, which is what the unified timeline is
supposed to be. It ranks second only because Purlin Co. holds USPTO registration 7303029 in
**Class 42 for AI software**, operates `purlin.com` and `purlin.app`, and has 40,000 users. The
ticket says trademark is informational, so this is your call, not a disqualification — but it is the
same shape of fact that killed `ritma`.

**Do not take Ricasso or Detent.** Ricasso has a pending Class 9 application for a mobile app and a
shipping app on both stores. Detent has three apps already using the exact name, one of them a
fitness tracker.

---

## Not verified

- **EUIPO / Rospatent.** No EU or Russian trademark search was performed for any candidate. The
  TMview API (`tmdn.org/tmview/api/search/results`) returned empty from this network, and Rospatent
  was not attempted. All trademark data above is USPTO only.
- **USPTO for Heddle, Orbin, Newel, Parrel.** Not checked — they were already ranked out.
- The USPTO query matched the literal string inside the wordmark field. It would not surface
  phonetic equivalents (`Muntyn`, `Sennitt`, `Purlyn`), and no common-law or US state trademark
  search was done.
- **Domain registration status.** Only HTTP reachability was established, per the ticket's
  instruction. "No DNS" is not proof a domain is unregistered — check RDAP or a registrar before
  assuming any is buyable.
- **RuStore completeness.** The backend endpoint used is the one behind the catalog UI, and it
  matches fuzzily. Zero exact-name hits is strong evidence, not proof, that nothing ships under a
  name there.
- **Google Play** was checked by scraping the public search page, not an official API. Absence of a
  result is weaker evidence than the App Store and RuStore checks.
- Whether `ricasso.app` is operated by THISISDISCO VENTURES LLC is inferred from matching subject
  matter, not confirmed.
