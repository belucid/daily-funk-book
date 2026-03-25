# By Region

```dataviewjs
const pages = dv.pages('"Daily Funk Project/Entries"');

const byRegion = {};
for (const page of pages) {
  const r = page.region || "(no region)";
  if (!byRegion[r]) byRegion[r] = [];
  byRegion[r].push(page);
}

const sorted = Object.keys(byRegion).sort();

for (const region of sorted) {
  const entries = byRegion[region];
  dv.header(3, `${region} (${entries.length})`);
  dv.table(
    ["Entry", "Artist", "Track", "Day", "Status"],
    entries
      .sort((a, b) => (a.file.name < b.file.name ? -1 : 1))
      .map(p => [p.file.link, p.artist, p.track, p.day, p.status])
  );
}
```
