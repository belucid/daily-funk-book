---
description: Verify and structure the streaming & purchase links for complete entries, then mark them verified for the funk.day website.
allowed-tools: Glob, Grep, Read, Edit, WebSearch, WebFetch, Bash(open *)
---
# Funk Entry Link Verification Assistant

## Purpose

Every entry that reaches `status: complete` has prose that is book-ready. This
command takes a complete entry the rest of the way to **web-ready**: it finds,
verifies, improves, and structures the entry's music-service links into the
**Structured:** JSON block in its Research Notes (see Where the data lives), then
advances the entry to `status: verified`.

Only `verified` entries get a page on the funk.day website (at `funk.day/<code>`).
The book PDF continues to include all `complete` (and `verified`) entries.

This is the companion to `/research` (`Research.md`). Where `/research` fills in
facts and *suggests* links in free-form Research Notes, `/verify` produces the
final, verified, structured link data the website renders.

## Your task: choose one entry to verify

This command verifies **one entry at a time**, interactively (the human must see
and confirm every link — see the cardinal rule). It is not a batch/parallel job.

When invoked:

1. Use `Grep` to find entries with `status: complete` under `./Entries/` (these
   are the entries ready to be verified — once verified, an entry leaves this
   list).
2. Present them as a numbered list: number, artist, track, and `funk.day code`.
3. Let the human choose **one**, accepting either:
   - a list number or filename, or
   - a 4-character `funk.day code` typed directly (e.g. `zk89`).
4. Read that entry and verify it end-to-end through the process below. Do not
   move on to another entry in the same run unless the human asks.

## The cardinal rule: the human verifies every link

**The agent does the legwork; the human confirms every link before it is locked
in.** The agent never writes a link to the structured block on its own authority.
For each service the agent:

1. Finds the best candidate link(s), preferring track-level over album-level.
2. **Proves** what each candidate resolves to, using the per-service method below
   (an oEmbed/API title, a fetched page title — never "I'm confident it's right").
3. Presents the candidate(s) and the proof to the human **and opens each
   candidate link in the human's default browser** by running `open <URL>` (one
   `open` per link). The agent's own oEmbed/API check is necessary but not
   sufficient — the human looks at the real page. Always both: list the links as
   clickable text *and* `open` them.
4. Waits for the human to confirm (or correct) before treating it as verified.

**Always `open` things for the human — never just paste a URL and ask them to go
there.** This applies to two cases:
- **Candidate links** to verify → `open <URL>` (step 3 above).
- **Search pages** for JS-walled services the agent can't scrape (YouTube Music,
  Amazon Music, etc.): when you need the human to find a link, `open` the
  service's search URL for them (e.g. `open "https://music.youtube.com/search?q=…"`)
  rather than printing it and asking them to navigate. Opening it is mandatory,
  not optional.

When the human has confirmed a service's links, write them into the **Structured:**
JSON block (see Where the data lives, below). Work through one entry's services as
a dialog, in the fixed service order below.

## Where the data lives

The structured link data does **not** live in the frontmatter (deep-nested YAML
trips up Obsidian's editor). It lives in the entry's Research Notes, in the
`### Streaming & Purchase Links` section, which has three subsections:

- **Streaming:** — human-readable bullet list of the streaming links.
- **Purchase:** — human-readable bullet list of the purchase links.
- **Structured:** — a fenced ` ```json ` block holding the verified link array.
  This is the canonical machine-readable data the website build reads.

`cover` is the one piece that stays in the YAML frontmatter (a single bare
filename — Obsidian is fine with that).

## Editing rules — strictly enforced

You may only write to:

1. The **Structured:** ` ```json ` block in the `### Streaming & Purchase Links`
   section (create the subsection and block if absent).
2. The **Streaming:** and **Purchase:** bullet lists in that same section — but
   only to **bring them into sync with the verified Structured data** (see
   below). Do not touch any other part of the Research Notes.
3. The `cover` field in the YAML frontmatter — the bare filename of the approved
   cover image (see Cover Art below). Create it if absent.
4. The `status` field — only the transition `complete` → `verified`, and only
   after every service has been resolved (found-and-confirmed or
   verified-absent), the cover is settled, and the human approves completion.

You must never modify the entry body, any other YAML field, or any other part of
the Research Notes.

Do not advance `status` to `verified` until the human has confirmed the full
service sweep for that entry.

## Keep the prose lists in sync with the Structured block

As each service is verified and written into the **Structured:** JSON, also
**update that service's lines in the Streaming:/Purchase: bullet lists so the two
match.** The bullet lists start life as `/research`'s unverified suggestions; by
the time the entry is `verified`, they must reflect the same final, verified URLs
and labels as the JSON — same links, same Live/Studio (etc.) labels. Fix a
changed URL, replace an album link upgraded to a track link, add a newly found
link, and mark a verified absence. The prose and the JSON are two views of one
verified truth; never leave them disagreeing.

**Order the prose lists by the fixed service order**, the same order as the JSON
array and the website render order (Spotify, Apple Music, YouTube Music, YouTube,
Amazon Music, Tidal, Qobuz, SoundCloud, Deezer; then the
buy services). Don't leave the bullets in `/research`'s original ad-hoc order —
re-sort them as you verify so prose and JSON read top-to-bottom the same way.

## The target services (fixed order)

Every entry is checked against all 11 services below, **in this order** — this is
also the render order on the website. For each service the outcome is either a
set of verified links or a verified absence (you genuinely confirmed the track is
not on that service, not merely "couldn't find it quickly").

| # | `service` id | Display name   | kind   |
|---|--------------|----------------|--------|
| 1 | `spotify`      | Spotify        | stream |
| 2 | `appleMusic`   | Apple Music    | stream |
| 3 | `youtubeMusic` | YouTube Music  | stream |
| 4 | `youtube`      | YouTube        | stream |
| 5 | `amazonMusic`  | Amazon Music   | stream |
| 6 | `tidal`        | Tidal          | stream |
| 7 | `qobuz`        | Qobuz          | stream |
| 8 | `soundcloud`   | SoundCloud     | stream |
| 9 | `deezer`       | Deezer         | stream |
| 10| `discogs`      | Buy on Discogs | buy    |
| 11| `amazonBuy`    | Buy on Amazon  | buy    |

The display name, brand color, logo, and `kind` live in the website's service
registry — do **not** store them per entry. The entry stores only the `service`
id and its verified links.

## Link selection rules

- **Prefer track-level links.** A link to the exact recording is best.
- **Album-level links are acceptable** when the service has no track-level URL for
  this recording (common on Apple Music, Tidal, Qobuz, Deezer). Do not *prefer*
  album links — only fall back to them. Label them honestly (e.g. `Live · album`).
- **Multiple links per service are allowed and expected.** A service may carry
  more than one verified link (e.g. Spotify Live + Studio; Discogs album + single;
  Amazon CD + Vinyl). Each gets its own `{ label, url }`.
- **Never infer version from a title.** Do not decide live vs studio vs edit from
  a track/page title or filename — titles are unreliable and inconsistent across
  services. A "studio"-looking title can be a live cut, and two different
  recordings can share an identical title. Establish version by the **human's
  album-grouped catalog view** and/or a **duration cross-check** against the
  entry's `track length`, never by the title string alone. (Burned us twice on
  `zk89`: a Spotify "studio" link that was a live reissue cut, and two YT Music
  catalog tracks with byte-identical titles.)
- **Label semantics:** labels are **not a codifiable taxonomy** — there is no
  generic label scheme to apply. A label exists only to disambiguate *which* link
  within a service, so:
  - **Single link → no label** (omit it). The website renders a single-link card
    as just the service name. **Most entries are single-link** per service.
  - **2+ links → label each**, with whatever distinction is **meaningful for this
    specific entry**. For `zk89` that's `Live version` / `Studio version`, because
    the live-vs-studio contrast is the point of this entry. Another entry's
    multiple links might be a different distinction entirely.
  - **Do not encode album-vs-track in the label.** Whether a given link happens to
    be album-level or track-level is irrelevant to the label (no "· album"
    suffix); use the same entry-specific label regardless of link granularity.
  - Use the **same labels across services** for the same distinction (so the Live
    link reads `Live version` on Spotify, Qobuz, Tidal, etc. alike).

## The Structured schema (JSON block)

The canonical data is a JSON **array of service objects**, pretty-printed inside a
` ```json ` fenced block under the **Structured:** subsection. Each object is
`{ "service": <id>, "links": [ { "label"?, "url" } ] }`.

```json
[
  {
    "service": "spotify",
    "links": [
      { "label": "Live version", "url": "https://open.spotify.com/track/1cQT5czRr73b52Y7AxZt3p" },
      { "label": "Studio version", "url": "https://open.spotify.com/track/0X4dm6ZioTcqYDUuhTSoqj" }
    ]
  },
  {
    "service": "appleMusic",
    "links": [
      { "url": "https://music.apple.com/us/album/live/537033638" }
    ]
  },
  {
    "service": "youtubeMusic",
    "links": [],
    "note": "No track- or album-level page found; only third-party uploads."
  }
]
```

- List services in the fixed order above.
- A single-link service **omits `label`**; a multi-link service labels every link.
- **Absences are recorded explicitly.** When a service is verified to have no link
  for this recording, include it with an **empty `links` array** (and optionally a
  `"note"` explaining how absence was confirmed). A later run then knows the
  absence was already verified rather than re-checking from scratch.
  - non-empty `links` → verified present; the website renders it.
  - `"links": []` → verified absent; the website renders nothing (its registry
    only shows services whose `links` array is non-empty).
  - service **missing from the array** → not yet checked.
- A fully verified entry has all 11 services represented in the array (each either
  with links or with `"links": []`).
- It must be valid JSON (double-quoted keys/strings, no trailing commas) — the
  build parses it directly.

---

# The process: Odesli first, per-service fallback

Finding the same recording on a dozen services by hand is slow. **Odesli
(song.link) is the shortcut — try it first for everything.** It maps one link you
already have to the same song on many other platforms, so the agent generates
candidates in bulk and the human just **verifies** (glance at opened tabs) instead
of searching service by service.

## Step 1 — Establish anchors (easy, agent-verifiable)

Find the links the agent can both *find* and *verify* on its own, for each version
(e.g. Live and Studio):

- **Spotify** — search + oEmbed (§1).
- **Apple Music** — iTunes search/lookup API (§2).

These become the Odesli anchors.

## Step 2 — Fan out through Odesli

For **each anchor URL** (each version), `WebFetch`:

```
https://api.song.link/v1-alpha.1/links?url=<anchor url>
```

Read `linksByPlatform`. Collect the urls for platforms in our 13-service list:
`spotify`, `appleMusic` (`itunes`), `youtube`, `youtubeMusic`, `amazonMusic`
(`amazonStore`), `tidal`, `deezer`, `soundcloud`. (Ignore platforms not in our
list — pandora, napster, boomplay, anghami, etc.)

**Union multiple anchors.** Coverage is anchor-dependent — Spotify and Apple
anchors return *different* sets (on `zk89`, the Spotify anchor gave Tidal+Deezer
but no Apple; the Apple anchor gave Apple but no Deezer). Run Odesli on every
anchor and merge.

**Normalize Odesli's urls** to our canonical forms (Odesli adds affiliate/region
cruft): Tidal `listen.tidal.com/track/{id}` → `tidal.com/browse/track/{id}`; Apple
`geo.music.apple.com/...?at=...&ct=...` → the clean `music.apple.com/...?i={trackId}`
(prefer the iTunes-API url from §2); Amazon → `music.amazon.com/tracks/{trackAsin}`;
strip `?si=`, `?ref=`, `&at=`, `&ct=`, `marketplaceId`, etc.

## Step 3 — Verify every candidate, then `open` for the human

Odesli is a *matcher*, not an oracle — its match can be the wrong version or
pressing (on `zk89` it produced **three different Tidal ids** for the live cut).
So for every candidate it returns:

1. Verify with the service's own method from the playbook below (Spotify oEmbed,
   iTunes API, YouTube oEmbed, Tidal page title, …) where one exists.
2. `open` it in the human's browser. The human confirms it is the right recording
   and the right version (live/studio) — **never write on Odesli's say-so alone.**

## Step 4 — Manual fallback for gaps

For any target service Odesli did not return (commonly **YouTube, YouTube Music,
Amazon Music, SoundCloud, Qobuz** — and the JS-walled ones
in general), fall back to that service's **per-service playbook** below to find
the link by hand, then verify + `open` as usual.

> The per-service playbook that follows is therefore dual-purpose: it is both the
> **verification method** for any candidate (Odesli's or manual) and the
> **manual finder** for services Odesli misses.

---

# Cover art

Each entry should have one cover image (the album/single artwork). It is gathered
**during** the service sweep: many services expose a grabbable cover, at different
resolutions and quality, so treat cover art as a by-product of verification rather
than a separate hunt.

## Process

1. **Collect candidates as you go**, but follow this source preference:

   - **Preferred: Apple Music / iTunes API.** High resolution and reliably the
     correct sleeve for the specific album. Take `artworkUrl100` from the
     `/lookup` or `/search` result and **upscale by editing the size segment** of
     the URL — replace the trailing `…/100x100bb.jpg` with `…/1200x1200bb.jpg`
     (Apple serves up to ~3000×3000). This is the default cover source.
   - **Caution: Spotify.** Capped at **640×640**, and — more importantly — its
     **per-track embedded artwork is not a reliable album identifier**. A track
     tagged "Live" can still carry the *studio* sleeve (observed on `zk89`: the
     Spotify Live track returned the *Everything Is Everything* studio cover, not
     the *Donny Hathaway Live* cover). Do not trust Spotify track art to tell you
     which album sleeve you are getting. Use only as a last resort, and verify the
     image visually.
   - **Avoid: Discogs.** Discogs images are usually user-uploaded photographs of
     physical copies (angled, lit unevenly, sometimes the wrong pressing). Not a
     good cover source — do not use it for `cover`.

   Prefer the sharpest source that shows the correct artwork for *this* recording;
   in practice that is almost always the upscaled Apple/iTunes artwork.
2. **Stage, don't commit.** Download a candidate to a temp staging path (e.g.
   `/tmp/funk-covers/<code>-<service>.<ext>`). Determine the real format with
   `file` — the URL usually has no extension; the bytes decide whether it is
   `.jpg`, `.png`, or `.webp`. **Never stage the file with a `.img` (or other
   non-image) extension, even as a placeholder** — on macOS `open` hands `.img`
   to DiskImageMounter instead of the image viewer, so the human sees nothing.
   Give the staged file its **real image extension** (`.jpg`/`.png`/`.webp`, per
   `file`) before you `open` it. If you must download first and check the bytes
   after, `mv` it to the correct extension *before* opening.
3. **Human verifies the image.** `open <staged-image>` so the human sees it in
   their default viewer, exactly as with links (the staged file must already carry
   its real image extension — see above). The human confirms it is the right
   artwork (and picks among candidates if there is more than one).
4. **Make it canonical only on approval.** Move the approved file to the funk.day
   site's covers directory — an **absolute path in the separate `funk-a-day`
   repo** (NOT inside this vault):

   ```
   /Users/sean/Source/funk-a-day/docs/assets/covers/<code>.<ext>
   ```

   Name it by the 4-char code; the extension is whatever the bytes are. Then write
   the **bare filename** (code + extension) to the entry's `cover` field:

   ```yaml
   cover: zk89.jpg
   ```

   The site `build` reads that filename and serves the image from that covers
   directory. The extension isn't known ahead of time, which is why the field
   stores the full filename, not just the code.
5. **No good cover?** If no service offers acceptable artwork, leave `cover` empty.
   The website's no-art layout handles this (single-column, no cover rail).

When documenting each service's playbook below, record **whether and how** that
service yields a cover (URL pattern, max resolution, format).

---

# Per-service verification playbook

This section is the durable record of *how* to verify each service, discovered
once so the agent does not re-derive it per entry. Follow the method for each
service exactly.

## 1. Spotify (`spotify`, stream)

**Supports track-level links — prefer them.** Spotify URL types:
`open.spotify.com/track/{id}`, `/album/{id}`, `/artist/{id}`, `/playlist/{id}`.
Use `/track/` for the specific recording; fall back to `/album/` only if the
track is not individually catalogued.

**Verification method — oEmbed (no auth, fetch-friendly):**
`WebFetch` the endpoint
`https://open.spotify.com/oembed?url=<the-spotify-url>` — it returns JSON
including a `title` field (the real track/album title) and a `thumbnail_url`
(cover art). The `title` is the proof of what the link resolves to. If oEmbed
returns an error or no title, the link is dead or wrong.

**Distinguishing versions:** Spotify appends **`- Live`** to live recordings;
the studio recording has no suffix. Use the oEmbed `title` to tell them apart and
to label them (`Live version` vs `Studio version`).

**Finding a link when the entry has none:** `WebSearch` for the track restricted
to Spotify (e.g. `site:open.spotify.com <artist> <track>`), take the candidate
`/track/` URL, then **verify it via oEmbed** before presenting it.

**Confirming absence:** if no track or album surfaces after searching, record
Spotify as verified-absent and note how you confirmed it.

**Cover:** the oEmbed `thumbnail_url` (640×640 via the `…0000b273…` size code) is
available but **unreliable for album identity** — see Cover Art above. Not the
preferred cover source.

**Worked example (entry `zk89`, Donny Hathaway – "Voices Inside"):**
- `…/track/1cQT5czRr73b52Y7AxZt3p` → oEmbed title `"Voices Inside (Everything Is Everything) - Live"` → **Live version** ✓
- `…/track/7fLbUdinXGkwjeH8j40seK` → oEmbed title `"Voices Inside (Everything Is Everything)"` (no suffix) → **Studio version** ✓
- Result: a genuine multi-link service.

```json
  {
    "service": "spotify",
    "links": [
      { "label": "Live version", "url": "https://open.spotify.com/track/1cQT5czRr73b52Y7AxZt3p" },
      { "label": "Studio version", "url": "https://open.spotify.com/track/0X4dm6ZioTcqYDUuhTSoqj" }
    ]
  }
```

## 2. Apple Music (`appleMusic`, stream)

**Supports track-level links — prefer them.** A track-level Apple Music URL is an
album URL with a track query param:
`music.apple.com/{country}/album/{album-slug}/{albumId}?i={trackId}`. An album-only
URL omits `?i={trackId}`. Use the `?i=` form for the specific recording; fall back
to the album URL only if the track isn't individually catalogued.

**Verification method — the iTunes API (no auth, JSON, fetch-friendly):**

- **Resolve a known id:** `WebFetch`
  `https://itunes.apple.com/lookup?id={id}` → JSON with `artistName`,
  `collectionName` (album), `collectionType`, `trackName`, `trackTimeMillis`,
  and the canonical `trackViewUrl` / `collectionViewUrl`.
- **Find track-level links:** `WebFetch`
  `https://itunes.apple.com/search?term={artist+track}&entity=song&limit=15`
  → results each carry `trackName`, `collectionName`, `trackTimeMillis`, and
  `trackViewUrl` (the canonical link). Pick the result whose `collectionName`
  matches the entry's intended album; use its `trackViewUrl`.

**Confirm the right recording with `trackTimeMillis`.** Convert to `m s` and check
it against the entry's `track length` / body. This is how live vs studio vs
compilation cuts are told apart (e.g. live ≈ 13m35s vs studio ≈ 3m29s).

**Clean the URL:** strip the trailing `&uo=4` (an iTunes affiliate tracking param)
the API appends to `trackViewUrl` / `collectionViewUrl`. Keep the `?i={trackId}`.

**Note on catalog metadata:** Apple's `trackName` spelling may differ slightly from
the entry (e.g. "Voice Inside" vs "Voices Inside"); the `trackId` is the source of
truth, not the displayed title.

**Finding when the entry has none:** use the `/search` endpoint above; if nothing
matches after searching, record Apple Music as verified-absent.

**Cover — this is the preferred cover source.** Take `artworkUrl100` from the
`/lookup` (album) or `/search` result and upscale the URL's size segment
(`…/100x100bb.jpg` → `…/1200x1200bb.jpg`, up to ~3000). For `zk89` the *Live*
album (`537033638`) yielded artwork `075678027222.jpg` at 1200×1188 → saved to the
covers dir (see Cover Art) as `zk89.jpg`.

**Worked example (entry `zk89`):** two track-level links found, both versions —
Live (`?i=537033681`, 815,080 ms ≈ 13m35s) and Studio (`?i=1018383889`,
209,000 ms ≈ 3m29s) → a multi-link service.

```json
  {
    "service": "appleMusic",
    "links": [
      { "label": "Live version", "url": "https://music.apple.com/us/album/voice-inside-everything-is-everything-live/537033638?i=537033681" },
      { "label": "Studio version", "url": "https://music.apple.com/us/album/voice-inside-everything-is-everything/1018383581?i=1018383889" }
    ]
  }
```

## 3. YouTube Music (`youtubeMusic`, stream)

**Supports track-level links — prefer them.** A YT Music song is
`music.youtube.com/watch?v={videoId}` (shares the video id with YouTube). The
catalog entries are auto-generated **"… - Topic"** channel uploads ("Art Tracks").

**Finding — the human searches; the agent cannot.** `music.youtube.com/search` is
a JavaScript app; `WebFetch` returns nothing useful, so the agent cannot scrape
YT Music search. General `WebSearch` is also an unreliable *finder* here (it
misses and misranks catalog entries). The method:

1. **The agent MUST `open` the search page for the human** — run
   `open "https://music.youtube.com/search?q={url-encoded artist track}"` via
   Bash. Do **not** just print the URL and ask the human to navigate there; always
   open it for them, exactly as with candidate-link verification. (URL-encode the
   query: spaces → `%20` or `+`.)
2. The human — who sees results **grouped by album** — identifies the canonical
   song(s) (e.g. the *Live* album track vs the studio track) and copies each
   share link (⋯ → Share → Copy link).
3. The agent verifies each link the human provides (below) and structures it.

**Verification method — YouTube oEmbed (no auth):** `WebFetch`
`https://www.youtube.com/oembed?url=https://www.youtube.com/watch?v={id}&format=json`
→ JSON with `title` and `author_name`. An `author_name` ending in **"- Topic"**
confirms a genuine catalog Art Track.

**Critical limit — oEmbed verifies authenticity, NOT identity.** Two different
catalog tracks (Live vs studio) can return a **byte-identical** `title` and
`author_name`. The agent therefore **cannot** tell which album/version a link is —
only the human's album-grouped YT Music view can. Confirm "- Topic", but rely on
the human for which-is-which. (Observed on `zk89`: the Live track `3gpN-SJD-VY`
and the studio track `iX3l21eEpW0` both report exactly
`"Voices Inside (Everything Is Everything)" / "Donny Hathaway - Topic"`.)

**Clean the URL:** strip the `&si=…` share-tracking param; keep `watch?v={id}`.

**Confirming absence:** if the human finds no catalog entry in YT Music search,
record verified-absent.

```json
  {
    "service": "youtubeMusic",
    "links": [
      { "label": "Live version", "url": "https://music.youtube.com/watch?v=3gpN-SJD-VY" },
      { "label": "Studio version", "url": "https://music.youtube.com/watch?v=iX3l21eEpW0" }
    ]
  }
```

## 4. YouTube (`youtube`, stream)

**Closely related to YouTube Music.** YouTube and YouTube Music share video ids;
the practical difference between the two services is mainly the **shape of the
URL** (`www.youtube.com/watch?v={id}` vs `music.youtube.com/watch?v={id}`). This
is the video site, so prefer a real video upload (official music video,
performance footage, or a canonical upload) when one exists — but **"- Topic" art
tracks are perfectly acceptable here too.** There is nothing wrong with using a
Topic track for the YouTube slot where that is the best (or only) good option;
the **same id can legitimately fill both the YouTube and YouTube Music slots**,
each with its own URL form.

**Policy (decided): official preferred, best available upload as fallback.**
Prefer an official music video / performance video / official-artist-or-label
channel upload. If none exists (common for vintage funk), accept the **single
best, most-canonical upload** per version — a fan upload or the "- Topic" art
track, whichever is best. The "best" judgment — views, audio quality, full length
vs clip — is the **human's**, made in YouTube; the agent surfaces verified
candidates and `open`s the top ones to choose among.

**Finding:** plain `WebSearch` works for YouTube (unlike YT Music). Gather
`youtube.com/watch?v={id}` candidates.

**Verification method — YouTube oEmbed (no auth):** `WebFetch`
`https://www.youtube.com/oembed?url=https://www.youtube.com/watch?v={id}&format=json`.

- **404 from oEmbed = a dead/removed video.** Cheap link-rot check — drop it.
- **`author_name` reveals provenance:** a real personal/curator name (e.g. "Alf",
  "BLUES AND SOUL") = unofficial fan upload; "… - Topic" = catalog art track
  (acceptable here too — see above); an official artist/label channel = canonical.
- Remember **titles do not prove version** (see link-selection rules) — confirm
  live vs studio by the human's listen / duration, not the title.

**Confirming absence:** if there is neither an official video nor an acceptable
fan upload, record verified-absent.

**Worked example (entry `zk89`):** no official video exists; only "- Topic" audio
(→ YT Music) and fan uploads. Human picked the best Live upload (Alf,
`lSdglxtW8Lg`) and the best Studio upload (EarpJohn album rip, `Ozi7gA9j5nY`).

```json
  {
    "service": "youtube",
    "links": [
      { "label": "Live version", "url": "https://www.youtube.com/watch?v=lSdglxtW8Lg" },
      { "label": "Studio version", "url": "https://www.youtube.com/watch?v=Ozi7gA9j5nY" }
    ]
  }
```

## 5. Amazon Music (`amazonMusic`, stream)

**Not the retail store.** Amazon Music streaming lives on `music.amazon.com` — do
**not** use `amazon.com/…/dp/{ASIN}` retail product pages here (those are physical
CD/vinyl/digital store listings → candidates for **Buy on Amazon**, #13). The
entry's `/research` "Amazon Music" suggestion is often a retail `dp/` link and is
usually wrong for this service.

**Supports track-level links — prefer them.** Canonical clean form:
`music.amazon.com/tracks/{trackAsin}`. Album form: `music.amazon.com/albums/{albumAsin}`.

**No agent verification is possible — this is the hardest service.**

- `music.amazon.com/*` is a JavaScript app → `WebFetch` returns only a "Tuning in"
  loading shell (no content).
- `amazon.com/dp/*` product pages → `WebFetch` gets **HTTP 500** (bot-blocked).
- There is **no public oEmbed/API** for Amazon Music.

So Amazon Music is **fully human-driven**: the agent **MUST `open`** the search
page for the human — run `open "https://music.amazon.com/search/{url-encoded query}"`
via Bash (don't just paste the URL and ask them to navigate). The human finds the
track(s) in their logged-in player and provides the link(s). **The human's in-app
confirmation is the verification** — there is no server-side check.

**Cleaning the URL (the agent CAN do this deterministically):** the Share button
yields a messy
`music.amazon.com/albums/{albumAsin}?marketplaceId=…&musicTerritory=…&ref=dm_sh_…&trackAsin={trackAsin}`.
Its `trackAsin` **is** the clean track ASIN, so convert any share link to the
canonical form: extract `trackAsin` → `music.amazon.com/tracks/{trackAsin}`.
(Navigating to the track in the app also shows this clean `/tracks/{ASIN}` form
directly.) Strip all `marketplaceId` / `musicTerritory` / `ref` params.

**Confirming absence:** if the human can't find it in Amazon Music, record
verified-absent.

**Worked example (entry `zk89`):** Live `tracks/B008CAB4AA` (from a share link's
`trackAsin`), Studio `tracks/B0045X6K5C`.

```json
  {
    "service": "amazonMusic",
    "links": [
      { "label": "Live version", "url": "https://music.amazon.com/tracks/B008CAB4AA" },
      { "label": "Studio version", "url": "https://music.amazon.com/tracks/B0045X6K5C" }
    ]
  }
```

## 6. Tidal (`tidal`, stream)

**Supports track-level links — prefer them.** Canonical form:
`tidal.com/browse/track/{id}`. (Share links use `tidal.com/track/{id}/u` and
Odesli returns `listen.tidal.com/track/{id}` — normalize all to
`tidal.com/browse/track/{id}`; strip the `/u` suffix.)

**Verification method — Open Graph via WebFetch (no auth).** Unlike Amazon Music,
Tidal is **not** a JS wall for verification: `WebFetch` on a track or album URL
returns the page title `"{Track} by {Artist} on TIDAL"`, enough to confirm
artist + track. (Tidal also has an `oembed.tidal.com` endpoint, but it returns
only embed dimensions — use the page title instead.)

**Finding:** the *tracklist* on an album page is JS-loaded (not in the fetched
HTML), and search does not index track pages, so the agent generally **cannot
find** the track id itself. Get it from **Odesli** (primary) or have the **human**
grab it from Tidal (fallback). Either way, the agent **verifies** the resulting
URL via the page title.

**Version caveat:** the title does not mark live vs studio, and the same recording
exists under multiple Tidal pressings (for `zk89` we saw track ids `16214014`,
`4266263`, and `68336275` all for the live cut). Confirm version with the human.

```json
  {
    "service": "tidal",
    "links": [
      { "label": "Live version", "url": "https://tidal.com/browse/track/16214014" },
      { "label": "Studio version", "url": "https://tidal.com/browse/track/4616714" }
    ]
  }
```

## 7. Qobuz (`qobuz`, stream)

**Album-oriented — album-level links are the norm, and that's fine** (as long as
it's the right album). Qobuz does not expose reliable public per-track URLs; its
shareable pages are albums: `www.qobuz.com/{locale}/album/{slug}/{albumId}`. Use
the **US locale** (`us-en`) for consistency; normalize away other locales
(`gb-en`, etc.).

**Verification method — WebFetch reads the full tracklist (no auth).** Fetching a
Qobuz album page returns artist, album, and the **complete track listing with
durations**. So you can confirm both that it's the right album *and* that the
target track is on it (matching duration). Strong verification — better than a
title-only check.

**Finding:** `WebSearch` surfaces `qobuz.com/.../album/...` pages (or use Odesli).
Beware multiple editions/remasters (for `zk89`: `0603497931194` plus
`0603497933518` and `0603497933501` "Edition Studio Masters" variants) — pick the
canonical standard album and confirm by tracklist.

**Worked example (entry `zk89`):** Live album `0603497931194` (track 8, 13:35) and
Studio album *Everything Is Everything* `0081227221669` (track 1, 3:25), each
confirmed by fetching its tracklist.

```json
  {
    "service": "qobuz",
    "links": [
      { "label": "Live version", "url": "https://www.qobuz.com/us-en/album/live-donny-hathaway/0603497931194" },
      { "label": "Studio version", "url": "https://www.qobuz.com/us-en/album/everything-is-everything-donny-hathaway/0081227221669" }
    ]
  }
```

## 8. SoundCloud (`soundcloud`, stream)

**Track-level by nature.** URLs are `soundcloud.com/{account}/{track-slug}`.

**Verification method — SoundCloud oEmbed (no auth):** `WebFetch`
`https://soundcloud.com/oembed?format=json&url={track url}` → JSON with `title`,
`author_name`, `author_url`. Confirms the track exists and its uploading account.

**Official-account tell:** when the uploads come from an artist account whose track
titles use the **official catalog metadata formatting** (e.g. the exact Rhino
Atlantic string "[Live at the Bitter End, New York City, 1971]"), the account is
almost certainly the official/label one. The profile page itself
(`soundcloud.com/{account}`) is **JS-walled** — `WebFetch` gets a "browser isn't
compatible" shell — so the agent **cannot** confirm the verified badge; the human
confirms official status in the browser.

**oEmbed titles are version-blind here too.** Multiple uploads on the same account
share the byte-identical oEmbed title `"Voices Inside (Everything Is Everything) by
Donny Hathaway"` regardless of which take they are — so the agent cannot tell
studio from live from alternate. The human listens and designates (see
link-selection: never infer version from a title).

**Worked example (entry `zk89`):** the `donnyhathaway` account hosts several takes
under near-identical slugs (`…-1`, `…-3`, `…-4`, `…-6`). oEmbed titles did **not**
distinguish them and two slugs whose titles *claimed* "[Live at the Bitter End…]"
were the wrong takes on listening. The human identified Live `…-4` and Studio
`…-1` by ear.

```json
  {
    "service": "soundcloud",
    "links": [
      { "label": "Live version", "url": "https://soundcloud.com/donnyhathaway/voices-inside-everything-is-4" },
      { "label": "Studio version", "url": "https://soundcloud.com/donnyhathaway/voices-inside-everything-is-1" }
    ]
  }
```

## 9. Deezer (`deezer`, stream)

**Supports track-level links — prefer them.** Canonical form:
`www.deezer.com/track/{id}` (drop any `/{lang}/` locale segment, e.g. `/en/`).

**Fully agent-driven — Deezer has a public no-auth API for both search and
lookup.** This is the cleanest of the "manual" services; the agent can find *and*
verify without a login:

- **Find:** `WebFetch`
  `https://api.deezer.com/search?q=artist:"{artist}" track:"{track}"` (add
  `album:"{album}"` to disambiguate). Each result carries `id`, `title`, `album`,
  `duration` (seconds), and `link`.
- **Verify:** match the result's `album` and `duration` against the entry. (Or
  look up a known id directly: `https://api.deezer.com/track/{id}`.)

**Confirm the right recording with `duration`.** Convert seconds → `m:ss` and check
against the entry's `track length` / body — this distinguishes live vs studio vs
alternate (live ≈ 815s/13:35 vs studio = 205s/3:25).

**Confirming absence:** if the API search returns no matching recording, record
verified-absent.

**Worked example (entry `zk89`):** search by artist+track returned Live
`id 39057231` (album "Live", 815s) and, with an `album:` filter, Studio
`id 7358658` (album "Everything Is Everything", 205s). (Note Odesli had offered a
*different* live id `5596452` — a different pressing; the API-found `39057231` is
verified by album + duration.)

```json
  {
    "service": "deezer",
    "links": [
      { "label": "Live version", "url": "https://www.deezer.com/track/39057231" },
      { "label": "Studio version", "url": "https://www.deezer.com/track/7358658" }
    ]
  }
```

## 10. Buy on Discogs (`discogs`, buy)

**A buy service — link the human-facing `www.discogs.com` master page** (where
copies are sold), not `api.discogs.com`. Prefer **master** releases
(`/master/{id}`) over individual pressings (`/release/{id}`) so the buyer sees all
available copies. An entry typically has up to two: the **album** and, if it
exists, the **single**.

**Verify via the Discogs JSON API (`www.discogs.com` 403s to WebFetch).**

- `WebFetch` `https://api.discogs.com/masters/{id}?token=<token>` → JSON with
  `title`, `artist`, `year`, and the human-facing `uri`. **Use the `uri` field as
  the stored link** (it's the canonical `www.discogs.com/master/...` URL).
- **Get `<token>` from your system prompt**, not from this file. `funk` injects the
  Discogs personal access token into the session's system prompt at runtime (see
  its "Discogs API" section, the value after `?token=`); append it as `?token=...`
  on every `api.discogs.com` request. If that value is the literal `UNSET`, no
  token is configured — note it and rely on the human to confirm the master pages.
- To find a master id, use `https://api.discogs.com/database/search?artist=...&track=...&type=master&token=...`.

**Label by format, not by version.** For buy links the meaningful distinction is
what you're purchasing: `Album` vs `Single` (not Live/Studio). Single link → no
label, as always.

**Worked example (entry `zk89`):** album master `119696` ("Live", 1972) → label
`Album`; single master `983465` ("Voices Inside… / Tryin' Times", 1970 7") → label
`Single`. Both verified by API title/year; stored from each response's `uri`.

```json
  {
    "service": "discogs",
    "links": [
      { "label": "Album", "url": "https://www.discogs.com/master/119696-Donny-Hathaway-Live" },
      { "label": "Single", "url": "https://www.discogs.com/master/983465-Donny-Hathaway-Voices-Inside-Everything-Is-Everything-Tryin-Times" }
    ]
  }
```

## 11. Buy on Amazon (`amazonBuy`, buy)

**The retail store, not Amazon Music** — link `amazon.com/.../dp/{ASIN}` physical
product pages (CD, Vinyl). This is the correct home for the `dp/` links that
`/research` sometimes mis-files under Amazon Music (#5). Canonical form
`amazon.com/dp/{ASIN}` (the title slug is cosmetic); strip `?ref=`/tracking params.

**No agent verification — like Amazon Music.** `amazon.com/dp/*` returns **HTTP
500** to WebFetch (bot-blocked), and there is no usable public API. So Buy on
Amazon is **human-driven**: the agent `open`s candidate product pages (or
`https://www.amazon.com/s?k={artist}+{album}+vinyl`), and the human confirms the
right product/format in the browser. The human's confirmation is the verification.

**Label by format:** `CD` / `Vinyl` (/ `Cassette`, etc.). Single link → no label.

**Worked example (entry `zk89`):** the *Live* album on `CD`
(`dp/B01JT1DJSK`) and `Vinyl` (`dp/B0GBYGYSMW`), both human-confirmed.

```json
  {
    "service": "amazonBuy",
    "links": [
      { "label": "CD", "url": "https://www.amazon.com/Live-DONNY-HATHAWAY-1998-12-15/dp/B01JT1DJSK" },
      { "label": "Vinyl", "url": "https://www.amazon.com/Live-Donny-Hathaway/dp/B0GBYGYSMW" }
    ]
  }
```
