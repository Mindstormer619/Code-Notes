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

----

## References
+ https://supadupaguides.medium.com/how-to-git-cherry-pick-only-changes-to-certain-files-359ab1e88e2