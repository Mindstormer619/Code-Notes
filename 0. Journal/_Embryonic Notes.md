# Embryonic Notes

```dataview
TABLE without id 
out AS "Embryo", file.link as "Origin", file.folder as "Folder"
FLATTEN file.outlinks as out
WHERE !(out.file) AND !contains(meta(out).path, "/")
SORT Folder ASC, meta(file.link).path ASC
```

# TODO Pages 

```dataview
TABLE
file.folder as "Folder"
FROM #todo
SORT file.folder ASC
```
