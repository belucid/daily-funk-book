# By Artist

```dataview
TABLE rows.file.link AS Entry, rows.track AS Track, rows.day AS Day, rows.genre AS Genre, rows.status AS Status
FROM "Daily Funk Project/Entries"
GROUP BY artist
SORT artist ASC
```
