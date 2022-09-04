# Miscellaneous/advanced Commands

 - merge --strategy=octopus ( when there are no conflicts) - merge 2 branches or more into current branch. suitable for cases where all branches working on different portion of repo, especially different directories(in this merge strategy , collisions are not allowed).
```shell
mkdir git-octopus-merge-demo
cd git-octopus-merge-demo
git init
 echo "initial work" >> initial.txt ; git add initial.txt ; git commit -m "initial commit"
for i in {1..3}; do git checkout main ; git checkout -b feature$i; mkdir dir$i; cd dir$i; echo  "file number $i" >> file$i.out; git add file$i.out; git commit -m "feature$i file" ; cd ..; done
git checkout main
git merge --strategy=octopus  feature1 feature2 feature3

```
 - git fetch from other remote repo(not origin) + checkout from it into worktree/index.
 - delete files temporarily, make operations without the files, and restore them from HEAD.
 - use reset --hard --soft --mixed in order to make mass updates of files with commits each iteration,  and resetting to checkpoints if there are mistakes and replay - show use-case with sed utility.
 
 - git add -p for staging selected hunks from file with edit hunk(e options in interactive menu)
 - then use git diff(implicit HEAD) --cached to see that only chunks selected are going to be added on next commit, this diff gives the difference between     ref and index.
```shell
   vi functions.java
   git add functions.java
   git commit -m "add functions.java"
   git status
   vi functions.java
   git add -p functions.java
   git status
   vi functions.java
   git restore --staged functions.java
   git status
   vi functions.java
   git add -p functions.java
   git status
   git diff --cached
   git diff
   git commit -m "only mature function commited"
   git status
   git diff
   ll
   git stash not mature function saving
   git stash save not mature function saving
   git stash list
   ll
   vi file1.out
   vi functions.java
   git stash list
   git stash pop


```
 - git diff (implicit HEAD) - only changes relative to index.
 - git diff two refs, git diff - compare between two refs
 - git diff REF path/to/file, compare file in worktree or index relative to same file in REF
 - git checkout -p, checkout chunks from file. 
 - Show git rm of external values file that shouldn't be part of the packed chart for packing helm chart and then reseting the working tree to HEAD after finishing packing helm charts.
 - commit --amend
