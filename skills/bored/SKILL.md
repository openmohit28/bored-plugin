---
name: bored
description: Recommend something to watch, read, or learn when the user is bored — novels, shows, movies, documentaries, a topic to learn (history, science, finance), tech-world content, or work upskilling (small lessons building a topic ladder). Builds a one-time taste profile on first run, then picks from it, and keeps two personal pages (Up Next shelf + Upskill Tracks) in sync. Use when the user says "I'm bored", "what should I watch/read", "recommend something", "/bored", "sync my tracks", or reports whether they liked something ("watched X, loved it", "dropped Y").
---

# Bored → recommendation

State lives in ONE file: `~/.claude/bored/taste-profile.md` (plain markdown).
Read it first, every time.

## 1. File missing → onboarding (once only)

Run this exactly once. Never re-run unless the user says "redo my profile".

The buckets below are only a starting point. Everyone's tastes are different, so
whatever they type in "Other" or in free text matters more than the options.

**Round A — broad strokes** (one `AskUserQuestion` call, all multiSelect,
max 4 options each; the tool adds "Other" automatically):
1. How do you like to spend free time? Watch (shows, movies, docs, anime) ·
   Read (books, comics/manga, articles) · Listen (podcasts, audiobooks) ·
   Play or do (games, courses, hands-on projects)
2. What pulls you into a story? Gripping (thriller, mystery, crime) ·
   Imaginative (sci-fi, fantasy, horror) · Human (drama, romance, slice of life) ·
   Light (comedy, feel-good)
3. Want learning picks too? About the world (history, science, culture, ideas) ·
   Skills for work · Hobbies and hands-on (cooking, music, a language, fitness, making things) ·
   No, just entertainment
4. Usual time when bored: <30 min · 1–2 hrs · a weekend binge · a multi-week book or series

**Then one free-text message** asking for:
- country, plus the languages they're happy watching or reading in (subtitles OK?).
  Country decides what's available where.
- streaming services / apps / libraries they have
- 3–5 all-time favourites of anything
- **curiosities**: any topics they've been meaning to learn about, from black holes
  and the Roman Empire to sourdough, chess, or how markets work. No list, their words.
- what they do for work (optional, only used for Skills picks)
- hard nos (gore, slow burns, sad endings, etc.)

**Round B — calibration.** Show ~20 well-known titles as a numbered list,
spread across their chosen formats and genres. Mix eras and countries, and
include titles that are popular in their country and languages, not only
English-language hits. Ask them to reply in one line, e.g. `1L 2D 5L 7- …`:
L = liked, D = disliked, `-` or omitted = not seen.
Seen titles teach more than stated preferences; weight them higher.

Write `~/.claude/bored/taste-profile.md`:

```
# Taste profile
## Profile          — formats, genres, country, languages, services, work, time budgets, hard nos
## Curiosities      — `- topic — why/where it came from — last picked YYYY-MM-DD`; grows over time
## Taste signals    — patterns inferred from likes/dislikes, marked (inferred)
## Seen             — `- Title (type) — liked|disliked|meh — one-line why, date`
## Recommended      — `- YYYY-MM-DD Title (type) — pending|liked|disliked|skipped`
## Upskill tracks   — one `### <topic>` per track (see step 4)
## Pages            — shelf: <url> · tracks: <url>   (filled by step 0)
```

Then set up the pages (step 0) and go straight to step 2 — they came here bored.

## 0. Pages (one-time setup)

Two pages ship in `assets/` next to this SKILL.md. Each user gets their own
private copy; never reuse someone else's URL (the db would be shared).

If `## Pages` is empty and the `Artifact` tool is available:
1. Load the `artifact-capabilities` skill, then publish
   `assets/up-next-shelf.html` (icon `bookmark`) and `assets/upskill-tracks.html`
   (icon `map`), each with `capabilities: {"db": {}}`.
2. Write both URLs into `## Pages`.
3. Seed the shelf: one `ArtifactData` batch into collection `items` with the
   onboarding Seen titles (status `done`, rating from their verdict).

No `Artifact` tool (not signed in to claude.ai, other client) → skip pages
entirely; the markdown profile works on its own. Say so once.

**Shelf** — collection `items`, one doc per title: title, type
(show|movie|novel|book|learning|video|podcast|anime|documentary|game|comic|audiobook|article|course;
  any other short lowercase type also works),
status (want|doing|done|dropped), genres[], moods[], rating (0-5),
score (/10 or null), length, where, note, addedAt (ms).

**Tracks** — collection `tracks`, one doc per track: name, color, order, blurb,
steps[{level,title,kind,length,url,why,status}].

## 2. Recommend

1. **Sync first** (if pages exist): `ArtifactData list items` and `list tracks`;
   copy any status/rating changes the user tapped into the profile (Seen,
   Recommended, Upskill tracks). The profile is the source of truth.
2. One quick `AskUserQuestion` (skip if they already said): mood
   (switch off / be gripped / learn something / laugh) and time available.
3. Give **3 picks**, different formats where possible:
   - **Safe bet** — closest match to Seen-liked titles
   - **Stretch** — adjacent genre/format they haven't tried
   - **Learn** — from Curiosities, whatever the domain. Rotate: pick the topic
     least recently picked, in a format they like (a doc for watchers, a book for
     readers, a hands-on first step for doers). Update its `last picked` date.
     If they said no learning picks, make this a **Wildcard** instead: something
     well loved outside their usual taste.
   Anything they mention wanting to know about, in any chat, goes into Curiosities.
4. Each pick: title, type, length, where to watch/read, and ONE line of
   "why you" tied to a specific title in Seen ("because you liked X").
   For channels, podcasts and blogs, name the exact video/episode/post and
   link it (verified via WebSearch), never just the channel.
5. Never recommend anything in Seen, Recommended, or the hard nos.
6. For "something new"/tech picks, use WebSearch for recent releases instead
   of relying on memory. Don't invent availability — say "check" if unsure.
7. If an Upskill track has a `pending` step, add it as an optional 4th line:
   "Up next on your <topic> track". Skip when mood is switch off.
8. Append the picks to Recommended as `pending`, and add them to the shelf
   (`items`, status `want`) in the same turn.

## 3. Feedback (any time)

When the user says they watched/read/dropped something: move it to Seen with
the verdict and their why, flip its Recommended status, update Taste signals
if it shifts a pattern, and update the shelf doc (status `done`/`dropped` +
rating) in the same turn.

"Update my profile" = edit the Profile section in place; show the change first.

## 4. Upskill tracks (any skill, work or hobby)

Lives in `## Upskill tracks` in the profile. One ladder per topic, whether it's
system design, Spanish, guitar, investing, or knife skills. Offer to start one
when a Curiosity keeps coming up or they ask to "get good at" something.

- **Start a track**: WebSearch for a well-rated, current starting point on the
  topic, tag its level (basics/intermediate/advanced), then add one step below and
  one above it. Every step is specific and verified: a link (video, article,
  lesson, course chapter) or, for hands-on skills, a concrete practice task
  ("cook a basic omelette, focus on heat control"). Under 30 min where possible.
- **Check basics** before stacking up: one `AskUserQuestion` (multiSelect,
  "which of these can you already do or explain?") with 3–4 concepts from the level below.
  Unticked → keep or add basics steps. All ticked → mark them `skipped-known`.
- **Step liked** → unlock the next `locked` step and add one new step at the
  level above. **Too easy / too hard / meh** → move sideways at the same level.
- **Page sync:** the user taps verdicts on the tracks page
  (`done-liked|done-meh|too-easy|lost-me|skipped-known`). On `/bored` or
  "sync my tracks": read `tracks`, copy changed statuses into the profile, apply
  the ladder rules, write new/unlocked steps back with `ArtifactData`
  (pin `if_version`). No republish needed for content changes.
- **New track** = new doc in `tracks` (next `order`, an unused color:
  blue/green/orange/magenta/violet/teal) + a `### <topic>` section in the profile.
- **Bored + learn mood with 2+ tracks:** show each track's next stop in one line and
  let the user pick which line to ride, instead of choosing for them.
