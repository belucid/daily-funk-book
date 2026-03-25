
### Getting Resources

  1. Discogs API token (most impactful)
  Discogs has a proper API. With a personal access token, https://api.discogs.com/masters/375941 works where the web URL 403s. You'd add the token to ~/.funk/config.toml, the funk script injects it into the
  system prompt, and Research.md instructs the agent to use API URLs instead of web URLs. Free account, no rate limit issues.

  2. Wikipedia REST API
  Instead of https://en.wikipedia.org/wiki/Cal_Tjader, use https://en.wikipedia.org/api/rest_v1/page/summary/Cal_Tjader. Free, no auth, returns clean JSON. Easy fix in Research.md.

  3. MusicBrainz API
  Free, open, no auth required. Good for release dates, artist info, recording data. Less rich than Discogs but never blocks. Worth adding as a standard source.

  4. Fallback instruction in Research.md
  For sites that can't be fixed (AllMusic also tends to block), instruct the agent to rely on WebSearch result snippets when WebFetch returns 403, and note the limitation explicitly rather than silently skipping.

---

### Generating book preview

Determining if there is too much content on a book page?
Generate PDF with current entries
Generate Website with current entries