# Reset

## Discarding Changes on a branch

You can stash them for later use (see below for stashing)

discard local changes permanently to a file:
```
git checkout -- <file>
```

Discard all local changes to all files permanently:
```
git reset --hard
```

Undoing a Hard reset
- Actually pretty easy, you just need to check out your reflog
```
git reflog
```
Output for this will generally look something like this (with most recent actions being higher to the top)
```
$ git reflog
1a75c1d... HEAD@{0}: reset --hard HEAD^: updating HEAD
f6e5064... HEAD@{1}: commit: added file2
```

You will see a list of the latest git actions. You can select where you want your codebase to be then reset it to that HEAD or that SHA1 identifier
```
git reset --hard f6e5064
# Or
git reset --hard HEAD@{1}
```