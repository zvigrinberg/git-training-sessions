# Juggle Between two features or more 

## Use case 1 - Feature not mature for commiting, need to put it a side and fix production bug.

 We're in feature/something branch, and want to save our current work, only to fix production bug and return to feature later:

1.First way(Stashing):

```shell
git stash save in middle of feature/something
git checkout main -b hotfix/fixBug
## fix bug in code
git add .
git commit -m "bug fix"
git push
## Open PR on Remote server
git checkout feature/something
git stash pop
## Continue working on feature/something.

```

2.Second way(temp commit, resetting to former commit and restoring from temp commit):

```shell
git add -u . 
git commit -m "temp commit"
git checkout main -b hotfix/fixBug
## fix bug in code
git add .
git commit -m "bug fix"
git push
## Open PR on Remote server
git checkout feature/something
git reset --hard feature/something~1
git restore --source=feature/something@{1} . --worktree / --staged
```

3.third way(temp commit, resetting and checkout paths from temp commit):

```shell
git add -u . 
git commit -m "temp commit"
git checkout main -b hotfix/fixBug
## fix bug in code
git add .
git commit -m "bug fix"
git push
## Open PR on Remote server
git checkout feature/something
git reset --hard feature/something~1
git checkout feature/something@{1} .
```

## Use case 2
