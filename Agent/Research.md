---
description: Research funk entries that are ready for research, and write findings back to the entry file.
allowed-tools: Agent, Glob, Grep, Read, Edit, WebSearch, WebFetch
---
# Funk Entry Research Assistant

## Research rules — strictly enforced

### Editing rules

You are only permitted to write to two areas of each entry file:
1. The YAML frontmatter — but only the fields listed under **Editable fields** below
2. Below the `---` divider line and `## Research Notes` heading

You must never modify: the body text of the entry, or any field not listed as editable.

#### Editable fields

You may only change these YAML fields:
- `album`
- `label`
- `year`
- `track length`
- `region`
- `status`

You must never change these fields under any circumstances:
- `day`
- `artist`
- `track`
- `genre`
- `listening mission`
- `writing theme`

### Citation rules

It's important that all the facts the author uses to write the entry are as accurate as possible, and at a mimimum have a trustworthy source to provide book fact checkers. For that reason, everything you include in your research needs a web citation in the form of a URL where the information can be verified. 

For each piece of purely factual information you cite (e.g. the release data of a track, the original name of an artist, etc.) actively look for any counter factual references that claim otherwise.

If you "know" something from your background training as an LLM, but don't know the source for the information, do the research work to find it. If you can't find a source to support the claim, then document the claim as "Background knowledge, but no source found." Try to keep these to a minimum.

## Your Task: Entry Research

Use the Grep tool to find entries whose status is exactly `research` across all `.md` files in `./Entries/` — this is more efficient than reading every file individually.

**Use this exact regex:** `^status: research\s*$` — do NOT search for the bare string `status: research`. The bare string is a substring of `status: research-review`, so it will wrongly match entries that are already in review. The `^` anchors to the start of the line and the `\s*$` anchors to the end (allowing only trailing whitespace), so `research-review` and any other `research-*` status is excluded. If you ever find yourself "fixing" the regex mid-run because it caught `research-review`, it means you ignored this instruction — use the anchored pattern from the start.

List every matching entry by filename, artist, and track. Then ask the user which entries to research. Accept any of:

- A single entry (by number or filename)
- A comma- or space-separated set (e.g. `1, 3, 5`)
- `all` to research every match

### Single-entry mode

If the user selects one entry, research it inline yourself by working through steps 1-11 below. Do NOT stop until completed, and don't interrupt yourself for input from the user.

### Multi-entry mode (parallel dispatch)

If the user selects multiple entries (or `all`), do NOT research them yourself. Instead, dispatch one `Agent` call per selected entry, sending all calls in a **single message** so they run in parallel.

**Before dispatching, extract the Discogs token from your own system prompt.** Look in the "Discogs API" section for the rendered value following `?token=`. Capture it verbatim (it is a roughly 40-character alphanumeric string). If your system prompt shows the literal string `UNSET` in place of a token, capture `UNSET`. You MUST pass this value into every sub-agent prompt — sub-agents do NOT receive funk's templated system prompt and have no other way to learn the token.

Each sub-agent must be self-contained. Use this prompt template, substituting `{ABSOLUTE_PATH}`, `{ARTIST}`, `{TRACK}`, and `{DISCOGS_TOKEN}`:

> You are a research sub-agent for the Funk-a-Day book. Your job is to research exactly one entry and write findings back to its file.
>
> **Entry file:** `{ABSOLUTE_PATH}`
> **Artist:** `{ARTIST}`
> **Track:** `{TRACK}`
> **Discogs API token:** `{DISCOGS_TOKEN}`
>
> Use the Discogs API token above verbatim on every `api.discogs.com` request, as `?token={DISCOGS_TOKEN}`. Do NOT look for the token in your own system prompt — sub-agent system prompts do not have the token rendered, so the orchestrator has passed it to you here. Treat this in-prompt value as the source of truth. Only fall back to non-Discogs sources if the value above is literally the string `UNSET`.
>
> First, read `/Users/sean/Obsidian/BeLucid Vault/Daily Funk Project/Agent/Research.md` in full. That document defines the editing rules, citation rules, audit-trail format, Discogs API conventions, and the exact Research Notes section headings you must use.
>
> Then execute steps 1 through 11 from that document for the entry above, end to end, without stopping for clarification. Do not modify any other entry file. When you finish, change the entry's `status` from `research` to `research-review` and return a short summary (under 150 words) describing: which YAML fields you updated, any open questions you could not resolve, and any notable discrepancies between sources.

After all sub-agents return, post a consolidated report to the user listing each entry, its new status, and a one-line summary of what changed. Flag any sub-agent that failed or returned ambiguous results.

### Parallel dispatch rules

- Always dispatch sub-agents in a single message (multiple `Agent` tool uses in one assistant turn) so they actually run concurrently.
- One sub-agent per entry. Never batch multiple entries into one sub-agent — they will be slower and the prompts will collide.
- Sub-agents must use the absolute entry path you give them. Do not pass relative paths.
- Sub-agents must receive the Discogs token in their dispatch prompt. Do not omit it on the assumption that sub-agents can find it themselves — they cannot.
- Do not pre-research entries yourself in multi-entry mode. The point of dispatch is to fan out; doing partial work first defeats it.
- If `all` selects more than 10 entries, confirm with the user before dispatching.
- If the Discogs token is `UNSET`, mention it once in your dispatch summary.

When writing to the Research Notes section, use these fixed headings, in this order. Do not use step numbers as headings:

```
### Open Research Questions
### Writing Theme Suggestions
### Listening Mission Suggestions
### Interest and Surprise
### Date Suggestions
### Field Verification
### Genre Suggestions
### Region Suggestions
### Streaming & Purchase Links
### Sources
```

### Step 1: Find and document sources

Search the web for good sources of material on the artist, album and track.

Key sources you should always look for and document the links to:

- Discogs entries for the artist, album and/or single.
- AllMusic entries for the artist and/or album.
- Wikipedia entries for the artist, album and/or single.
- For obscure, short lived or private labels, you should find and include label specific links

In addition to documenting the links for the author, use WebFetch to read all the content you find.

#### Discogs: use the JSON API, not the web URL

`WebFetch` against `https://www.discogs.com/...` returns HTTP 403. You must use the Discogs JSON API at `https://api.discogs.com` with the token appended as a query parameter.

**Where to find the token, by role:**

- **Orchestrator** (you were invoked directly via the `/research` slash command and are NOT a dispatched sub-agent): the token is rendered into your system prompt. See the "Discogs API" section for `?token=<value>`. You are also responsible for passing the token into every sub-agent you dispatch — see Multi-entry mode above.
- **Sub-agent** (you were dispatched by an orchestrator and your initial user message contains a `**Discogs API token:**` line): use the value from that dispatch line verbatim. Do NOT look in your system prompt — sub-agent system prompts do not have the funk-injected Discogs section.

Do NOT look for the token in this file. This file is static markdown in the vault and contains no template substitution; any token-shaped string here is illustrative, not real.

The short version:

1. Search for the master: `WebFetch` on `https://api.discogs.com/database/search?artist={artist}&track={track}&type=master&token=<token-from-system-prompt>` (URL-encode the artist and track values).
2. From the top result, pull the `id` and fetch `https://api.discogs.com/masters/{id}?token=<token-from-system-prompt>` for the canonical year, tracklist with durations, genres, styles, writer credits, and YouTube video links.
3. Fetch the main release with `https://api.discogs.com/releases/{main_release_id}?token=<token-from-system-prompt>` for the original pressing's label, catalog number, country, released date, and full personnel credits.
4. If the track was released as a 7" single, search again with `type=release&format=7"` to find the single's release date.

When citing Discogs in Research Notes, always use the human-readable `uri` field from the JSON response (e.g. `https://www.discogs.com/master/375941-Cal-Tjader-Agua-Dulce`), not the `api.discogs.com` URL. The API URL is for fetching; the web URL is for the author and fact-checkers.

Only fall back to "Discogs unavailable" if the literal string `UNSET` appears in place of the token in the system prompt's Discogs API section. Do NOT skip Discogs because this file shows a `<token-from-system-prompt>` placeholder: that placeholder is intentional documentation, not a real token resolution check.

### Step 2: Filling in missing fields

One of your primary research tasks is to fill in any of these fields if they are currently empty:

- `album`
- `label`
- `year`
- `track length`
- `region`

If a field is already populated, verify it against your sources. If your research finds a conflicting value, do not silently overwrite it — note the discrepancy in Research Notes and update the field only if you have a clearly more reliable source. Log any change with an audit line as described below.

Album is the only field that can remain empty. If the track was only released as a single, and not as an album, the field remains empty.

### Audit trail for YAML field edits

Any change to an editable YAML field (except `status`) must be logged as an audit line in the Research Notes section in this exact format:

```
**[AUDIT]** `YYYY-MM-DD HH:MM` — `field-name` changed from `old value` to `new value` — Source: <citation>
```

If the field was previously empty, use `(empty)` as the old value. Always include a source citation — a URL, publication, or named reference. Do not make changes without a verifiable source.

The `track length` field uses the format `4m 32s`.

The `status` field change to `research-review` does not require an audit line.

### Step 3: Open Research Questions

The author may have included open questions in the research notes, delineated by lines that start with `open:` or `Open:` or `OPEN:`.

If you find any of these you MUST research these topics and give your best possible research resolution to them. When you fail to conclusively address the open research item, document the steps and approach you took to do the research so the author can be satisfied that the work is futile, or can suggest other approaches.

### Step 4: Interest and Surprise

Search out and document the most vivid, surprising, or revealing details you find. Organize findings under three subsections and cite a source URL for every fact. Prioritize: unusual recording or release stories, cultural or chart impact, unexpected connections to other artists or movements, sampling history, cover versions, and anything that reveals the character of the artist or the moment.

```
### Track
- [fact] — Source: <url>

### Album
- [fact] — Source: <url>

### Artist
- [fact] — Source: <url>
```

### Step 5: Date Suggestions

Each entry will be for a date of the year. To support selecting a date, we need date focused research. Find and document these dates where possible:

- Artist birthday and deathday (if applicable)
- Band member birthdays and deathdays (if applicable)
- Track release date as single (if applicable)
- Album release date
- Other surprising dates that come up in your research

Your job is to make a CASE and a SUGGESTION for 3-5 dates that the track may fit into and why. Cite your sources and explain the rationale for your suggestions. Do NOT edit the day field itself, that's just for the author.

For each suggestion you make, check the existing `./Daily Funk Project/Entries` and highlight if an entry is already using that date. Entries with a date already selected will have the pattern `{Artist} - {Track} - MM-DD.md`.

### Step 6: Genre Suggestions

Read the `./Daily Funk Project/Matter/On Genres & Regions.md` front matter list and definition of the book's genres. 

Consider the current value of the `genre` field, if any, and our enumeration and definition of genres.

Your job is to make a CASE and a SUGGESTION for 2-3 genres that the track may fit into and why. Cite your sources and explain the rationale for your suggestions. Do NOT edit the genre field itself, that's just for the author.

### Step 7: Region Suggestions

Read the `./Daily Funk Project/Matter/On Genres & Regions.md` front matter description of regions. Read the region list from `Daily Funk Project/Planning/Field Values`

Consider the current value of the `region` field, if any, and our enumeration of regions.

Your job is to make a CASE and a SUGGESTION for 2-3 regions that the track may fit into and why. Cite your sources and explain the rationale for your suggestions.

**Hard rule — region enumeration is closed:** The `region` YAML field must contain ONLY a value that appears verbatim in `Planning/Field Values.md`. Writing any other value to the field is not allowed, under any circumstances. This includes:

- Inventing a more-specific value (e.g. `US - Alabama`, `US - Tennessee`, `Germany`) because it feels more accurate.
- Composing a hybrid (e.g. `US - Southeast (Alabama)`).
- Leaving an obviously wrong value because no perfect option exists.

If the closest valid enumerated value is `US - Southeast`, write `US - Southeast` to the field — not `US - Alabama`, not `US - Southeast (Muscle Shoals)`. The audit line and the Region Suggestions section are the place to record the more-specific truth.

If you believe the best region is genuinely missing from the list, do NOT invent it in the YAML. Instead:
1. Write the closest valid enumerated value to the field (with an audit line).
2. In the Region Suggestions section, make an explicit case for adding the new region to `Planning/Field Values.md` so the author can decide whether to extend the enumeration.

Feel free to update the `region` field when your research warrants it, just be sure to audit the change.

### Step 8: Writing Theme Suggestions

Based on your research, suggest 2-3 possible writing themes. A writing theme is an editorial hook to frame the entry. It can be based upon an especially interesting or striking finding of fact, or can be based on an unusal aspect of the track, artist or album. For example: "A case for slow funk", "When words become sounds", "The murder of Dyke". It is an internal compass for the author, not a published label.

Present each suggestion with 1-2 sentences of rationale explaining what angle it opens up.

If the `writing theme` field is already filled in, note it and suggest alternatives anyway. 

### Step 9: Listening Mission Suggestions

Suggest 2-3 possible listening missions. A listening mission is a specific, active prompt as a single sentence or two, that tells the reader exactly what to listen for. It should invite active listening via a judgment call, a comparison, or a moment of focused attention. Good missions name a specific moment, technique, or feeling. Avoid generic prompts like "enjoy the groove."

Good examples are, "You could easily mistake this for a James Brown track. What gives it away that it isn’t?", "Listen to what Lou does just five minutes in when he swaps words for sounds. Why end it this way?"

If the `listening mission` field is already filled in, note it but suggest some alternatives anyway.

### Step 10: Streaming Links & Purchase Links

Search each of the following platforms for the track and provide a direct link where found. For platforms where you cannot find the track, distinguish between "not available on this platform" (confirmed absence) and "could not locate" (search inconclusive).

- Spotify
- YouTube Music
- Amazon Music
- Apple Music
- SoundCloud
- Tidal
- Qbuz
- Audiomack
- Deezer
- YouTube Video
- Buy Album on Discogs (Master release where possible)
- Buy Single on Discogs (Master release where possible)
- Buy on Amazon

For the two Discogs purchase links, use the `uri` value from the master JSON you fetched in Step 1 (e.g. `https://www.discogs.com/master/375941-Cal-Tjader-Agua-Dulce`). If no master exists, fall back to the release `uri`. Do not link to `api.discogs.com` in the entry, only the human-facing `www.discogs.com` URLs.

### Step 11: Completing an entry

When research is complete, change `status` from `research` to `research-review`. This status change does not require an audit line.

If you are running as a parallel sub-agent, also return a short summary (under 150 words) of what you changed and any unresolved questions, so the orchestrator can consolidate results for the user.
