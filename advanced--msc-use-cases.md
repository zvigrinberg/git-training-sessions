# Miscellaneous/advanced Commands

 - merge --strategy=octopus ( when there are no conflicts) - merge 2 branches or more into current branch. suitable for cases where all branches working on different portion of repo, especially different directories(in this merge strategy , collisions are not allowed).
```shell
mkdir git-octopus-merge-demo
cd git-octopus-merge-demo
git init
# Creating initial work on main branch.
 echo "initial work" >> initial.txt ; git add initial.txt ; git commit -m "initial commit"
 
 #Creating 3 features branches, each one with its own directory and file.
for i in {1..3}; do git checkout main ; git checkout -b feature$i; mkdir dir$i; cd dir$i; echo  "file number $i" >> file$i.out; git add file$i.out; git commit -m "feature$i file" ; cd ..; done

#Merging feature1,feature2,feature3 into main
git checkout main
git merge --strategy=octopus  feature1 feature2 feature3

```
 - git fetch from other remote repo(not origin) + checkout from it into worktree/index.
```shell
  git clone git@github.com:RHEcosystemAppEng-Temenos/infinity-helm-chart.git
  cd infinity-helm-chart
  git fetch git@github.com:RHEcosystemAppEng-Temenos/temenos-ocp-deploys-automation.git main
  git checkout FETCH_HEAD infinity-ms/injections
  # removing checked out directory from index and move it to be untracked, that way we can work in the directory with the new fetched directory but 
    after we disconnected it from the git repo by tell it not to track it.
  git reset --mixed.
 ``` 
  - Use git reset --soft to "cancel" commits but save your current work on index and worktree 
  __*Example:*__
  ```shell
  #for example, you have index and worktree with new files that you want to retain , and you want to remove last commit.  
  # build repo and initial commit
   mkdir reset-soft-demo
   cd reset-soft-demo/
   git init
   echo "bla bla" > initialwork.out
   git add initialwork.out
   git commit -m "initial commit"
   git status
   git commit --help
   git log
   echo "bla bla 2" >> initialwork.out
   git commit -am "second commit"
   echo "bla bla new" >> newwork.out
   echo "bla bla infra" >> infrawork.out
   ll
   git status
   # New files that we want to keep work on it after discarding latest commit.
   git add infrawork.out
   git add newwork.out --intent-to-add
   git status
   git log
   #eliminate last commit(with commit message "second commit") by rewinding HEAD 1 commit back, but keep worktree and index of new files.
   git reset --soft HEAD@{1}
   git log
   git status
   git diff --cached
   
   # Please pay attention that we repositioned HEAD to be on previous commit, this reconstruct the state before commiting commit "second commit", and 
   # bring initialwork.out from this commit into index, so in order to discard it from index and then from worktree, just type.
   git restore --staged --worktree initialwork.out

  ```
 - delete files temporarily, make operations without the files, and restore them from HEAD.
 - use reset --hard in order to make mass updates of files with commits each iteration,  and resetting to checkpoints if there are mistakes and replay - show use-case with sed utility.
 
 - git add -p for staging selected hunks from file with edit hunk(e options in interactive menu)
 - then use git diff(implicit HEAD) --cached to see that only chunks selected are going to be added on next commit, this diff gives the difference  
   between ref and index.
```shell
   vi functions.java
   # Add some dummy functions definition in java
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
