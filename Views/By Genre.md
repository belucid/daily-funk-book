# By Genre

```dataviewjs
const targets = {
  "Classic Funk": 60,
  "Jazz-Funk": 60,
  "Soul Jazz": 40,
  "Deep Funk": 40,
  "Acid Jazz": 20,
  "P-Funk / Psychedelic Funk": 20,
  "Funk Rock": 20,
  "Minneapolis Sound": 20,
  "Afrobeat / Afrofunk": 20,
  "Latin Funk": 20,
  "Free Funk": 3,
  "Go-Go": 3,
  "Country Funk": 3,
  "Gospel Funk": 3,
  "Avant-Funk": 3,
  "Electro Funk": 3,
  "Nu-Funk": 3
};

const tiers = [
  { name: "Focus", min: 60, genres: ["Classic Funk", "Jazz-Funk"] },
  { name: "Substantial", min: 40, genres: ["Soul Jazz", "Deep Funk"] },
  { name: "Coverage", min: 20, genres: ["Acid Jazz", "P-Funk / Psychedelic Funk", "Funk Rock", "Minneapolis Sound", "Afrobeat / Afrofunk", "Latin Funk"] },
  { name: "Some Coverage", min: 3, genres: ["Free Funk", "Go-Go", "Country Funk", "Gospel Funk", "Avant-Funk", "Electro Funk", "Nu-Funk"] },
];

const pages = dv.pages('"Daily Funk Project/Entries"');

const byGenre = {};
for (const page of pages) {
  const g = page.genre || "(no genre)";
  if (!byGenre[g]) byGenre[g] = [];
  byGenre[g].push(page);
}

for (const tier of tiers) {
  dv.header(2, `${tier.name} — target: ${tier.min}+ each`);
  const rows = tier.genres.map(genre => {
    const entries = byGenre[genre] || [];
    return [genre, entries.map(p => p.file.link), `${entries.length} / ${targets[genre]}`];
  });
  dv.table(["Genre", "Entries", "Progress"], rows);
}

const ungenred = byGenre["(no genre)"];
if (ungenred && ungenred.length > 0) {
  dv.header(2, "No Genre Assigned");
  dv.table(["Entry", "Artist", "Track"], ungenred.map(p => [p.file.link, p.artist || "", p.track || ""]));
}
```
