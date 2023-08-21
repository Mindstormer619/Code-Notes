---
created: 2023-08-21 13:02
alias: 
---
#### Summary
+ 

----
# Git Cherrypick or Revert specific files

```shell
git cherry-pick -n $commit # works with revert also

# Unstage everything
git reset HEAD

# Stage what you want to keep
git add $path

# Make the work tree match the index, i.e. the "unstaged" files will now get reverted
git checkout .
```

Currently I have a bug with Obsidian/Syncthing where it randomly deletes nearly all my notes. When that happens, I need to find the specific commit that deletes all the notes and revert the deletes. The commit _may_ have additional changes made in some files, or new files added, which _shouldn't_ be reverted.

----

## References
+ https://supadupaguides.medium.com/how-to-git-cherry-pick-only-changes-to-certain-files-359ab1e88e2