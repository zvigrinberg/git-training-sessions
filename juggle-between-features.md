# Juggle Between two features or more/Work in parallel on two features

## Background

 Sometimes there is a need to work on at least 2 features/hotfixes in parallel, but at least one of them is not mature yet for commiting and pushing to the remote git server, and off course you don't want one feature to get in the way of the other one(and vice a versa).

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

4.fourth way(temp commit, resetting and cherry pick temp commit using git cherry-pick -n commitId).

## Use case 2 - Need to put a side a feature that is not mature for progressing with work(requirements not clear) but part of it is good, and move to another feature, continue with it, it's good but not complete, but in the meantime, a release branch should be closed before delivery urgently, and you must put it aside as well.

```shell
##on feature1
git stash save feature1 initial temp work.
git checkout feature2
## work on feature2 and add to index or to worktree.
git stash save feature2 progressive work.
git checkout release
## handle release branch' merges and testing ....
git checkout feature2
git stash list
## according to the stash that show description containing feature2, pick the stash number, in this case:
git stash apply stash@{0}
## continue to work on feature2 , do add, commit and push.
git checkout feature1
## according to the stash that show description containing feature1, pick the stash number, in this case:
git stash apply stash@{1}
## continue to work on feature1 , do add, commit and push.
.....

```

Note: When finishing with the work on feature2 and with the work of feature1, it's recommended to clear the stash list:
```
git stash clear
```

