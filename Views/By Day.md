
```dataviewjs
const pages = dv.pages('"Daily Funk Project/Entries"');
const scheduled = pages.filter(p => p.day != null && p.day != "");
const unscheduled = pages.filter(p => p.day == null || p.day == "");

dv.header(2, `Scheduled — ${scheduled.length} / 366`);
dv.table(
  ["Entry", "Day", "Artist", "Track", "Genre", "Status"],
  scheduled.sort(p => p.day).map(p => [p.file.link, p.day, p.artist, p.track, p.genre, p.status])
);

dv.header(2, `Unscheduled — ${unscheduled.length}`);
dv.table(
  ["Entry", "Artist", "Track", "Genre", "Status"],
  unscheduled.sort(p => p.file.name).map(p => [p.file.link, p.artist, p.track, p.genre, p.status])
);
```
