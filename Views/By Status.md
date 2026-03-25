# By Status

```dataviewjs
const pages = dv.pages('"Daily Funk Project/Entries"');

const stub            = pages.filter(p => p.status === "stub");
const research        = pages.filter(p => p.status === "research");
const researchReview  = pages.filter(p => p.status === "research-review");
const draft           = pages.filter(p => p.status === "draft");
const review          = pages.filter(p => p.status === "review");
const complete        = pages.filter(p => p.status === "complete");

dv.header(2, `Stub — ${stub.length}`);
dv.table(
  ["Entry", "Artist", "Track", "Genre"],
  stub.sort(p => p.file.name).map(p => [p.file.link, p.artist, p.track, p.genre])
);

dv.header(2, `Research — ${research.length}`);
dv.table(
  ["Entry", "Artist", "Track", "Genre"],
  research.sort(p => p.file.name).map(p => [p.file.link, p.artist, p.track, p.genre])
);

dv.header(2, `Research Review — ${researchReview.length}`);
dv.table(
  ["Entry", "Artist", "Track", "Genre"],
  researchReview.sort(p => p.file.name).map(p => [p.file.link, p.artist, p.track, p.genre])
);

dv.header(2, `Draft — ${draft.length}`);
dv.table(
  ["Entry", "Artist", "Track", "Genre"],
  draft.sort(p => p.file.name).map(p => [p.file.link, p.artist, p.track, p.genre])
);

dv.header(2, `Review — ${review.length}`);
dv.table(
  ["Entry", "Artist", "Track", "Genre"],
  review.sort(p => p.file.name).map(p => [p.file.link, p.artist, p.track, p.genre])
);

dv.header(2, `Complete — ${complete.length} / 366`);
dv.table(
  ["Entry", "Artist", "Track", "Day", "Genre"],
  complete.sort(p => p.day).map(p => [p.file.link, p.artist, p.track, p.day, p.genre])
);
```
