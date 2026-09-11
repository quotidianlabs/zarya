# Final product name

Type: research
Status: resolved
Blocked by: -

## Question

`zarya` is a working name chosen under time pressure. Settle the real one.

**Do not re-vet the eight names already killed.** Prior work, with reasons:

| Name | Verdict | Killed by |
|---|---|---|
| `stint` | usable | rejected on phonetics - reads as "stink" |
| `zarya` | usable | current working name; Overwatch owns EN search, Soviet-heritage feel in RU, ZARA proximity |
| `pomidor` | weak | grocery noun; category owns it informally; vulgar RU plural sense |
| `tempa` | dead | "Tempa: ADHD Daily Planner" shipped to App Store Productivity 2026-08-28, live in RU |
| `kolo` | dead | homophone of ко́ла (Coca-Cola) in RU; US Class 9 + 42 marks |
| `ritma` | dead | ritma.org is a habit tracker with streaks and heatmaps - the same product |
| `spira` | dead | three Spira habit apps launched 2026, one by the Fabulous publisher |
| `sutra` | dead | Kama Sutra owns RU App Store search; "tomorrow" in Serbo-Croatian |
| `kairo` | dead | a Flutter pomodoro/task app already ships as Kairo; = Cairo in 12 languages |
| `diurna` | dead | contains `урна` - trash bin in Russian |
| `taka` | dead | Swahili for trash; Bangladeshi currency; Polish/Bulgarian function word |

**What that history teaches, and what the search should therefore optimise for:**

- Four of the eight died on **category collision**, not linguistics. Habits/focus/pomodoro is an
  extremely crowded namespace and short evocative Latinate words are what everyone reaches for.
- The survivors survived by being *unpoetic*: `stint` because it is English-specific and
  semantically precise, `zarya` because it means nothing at all in Latin script.
- Two died on **embedded substrings in Russian** that no amount of taste would have caught.

Generate a fresh shortlist under these constraints, then vet each:

1. Reads clean in **EN and RU** - no bad substring, no vulgar homophone, no accidental pun
2. Says it out loud without landing on something ugly (the `stint`/`stink` failure)
3. Not a common noun or function word in any major language
4. Not a currency, city, or national brand
5. **Nothing shipping under it in Productivity on App Store, Google Play, or RuStore**
6. Not an inflected/case form of a real word (the `ritma`/`tempa` failure)
7. Check trademark exposure in Class 9 and Class 42, and domain availability

Note: the personal-tool bar means trademark risk is informational, not blocking. Category
collision and RU readability are the real filters.

## Answer

Shortlist and full evidence: [research/05-final-product-name.md](../research/05-final-product-name.md).

**Method.** The first three candidates generated in the style of the prior kill-list (`velmo`,
`talvo`, `nimbo`) were all already taken, two by habit/affirmation apps - confirming this
ticket's own lesson. The pool was rebuilt from **obscure concrete trade nouns** (joinery,
roofing, horology, rope work, weaving): the `stint` profile of English-specific, semantically
precise and unpoetic. ~25 screened, 10 vetted in depth against machine-readable sources - Apple's
iTunes Search API in the US *and* RU storefronts, RuStore's own backend queried in both Latin and
Cyrillic, scraped Google Play, the GitHub search API, Wiktionary's full language-section list per
candidate, and a ~90-item Russian bad-substring scanner.

**Recommended: `muntin`** - the bar that divides a window into panes.

- Zero exact-name apps on all three stores, in both languages
- Zero relevant GitHub projects
- **Zero USPTO wordmarks in any class**
- `мунтин` has no Russian entry, no bad substring, no brand shadow - **a cleaner Russian record
  than `zarya`**

Costs: English speakers will mistype it as *mountain*; it is a Catalan verb form (violates the
"not an inflected form" rule, though in neither target language); and one small unregistered
brand, muntin.digital, uses the word.

**Runner-up: `purlin`** - the better-sounding word with the better metaphor and an identically
clean store record. Ranked second because **Purlin Co. holds a live USPTO Class 42 registration
for AI software**, operates purlin.com and purlin.app, and has 40,000 users. Trademark is
informational at the personal-tool bar, so this is an override that can be made knowingly - but
"already a live software brand holding the .com" is the same shape of fact that killed `ritma`.

**The Russian substring scanner earned its keep again**, killing five candidates a non-Russian
speaker would have shipped: `reglet` and `astragal` (both real Russian nouns), `pawl` (reads as
`пол`), `collet` (one letter from `колет`), `soffit` (`софит`), and `mandrel` - which
transliterates to `мандрель`, containing `манд-`. That is the `diurna`/`урна` failure exactly,
and only the scanner caught it.

**`kerf` was killed by an app released four days ago**: *Kerf: Calm Routine Planner*, Productivity
and Lifestyle, released 2026-09-07, "build habits at your own pace". This namespace is being
consumed in real time.

Unverified: trademark is USPTO-only - EUIPO and Rospatent were unreachable. Domain *registration*
status was not established, only HTTP reachability.

**The pick itself is the human's.** This ticket surfaced the field; it does not choose.
