```dataview
LIST
FROM #moc
SORT file.name asc
```

```dataview
TABLE topic AS Topic, file.cday AS Date
FROM #source
WHERE status = "Reading"
```

```dataview
TABLE topic AS Topic, file.cday AS Date
FROM #source
WHERE status = "Processing"
```

