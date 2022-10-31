
# Present both rebase and merge conflicts.

- Create a conflict on purpose.
- show how to merge it using regular merge
- show to to resolve conflicts using git checkout --theirs and git checkout --ours.
- show how to resolve conflict using git mergetool(after conflict introduced)
- show to to do in advance git merge --strategy=recursive -X ours/theirs if all conflicts should be settled from one side only.
- show how to resolve conflicts after rebase using checkout --theirs --ours, git mergetool.
- show history log of merge after conflict resolved and merge commit recorded, and show how to avoid the merge commit by using git merge --squash or --no-commit

## Procedure

1. Create an empty repo , create several commits, and then a new branch, and create a conflict on purpose:
```shell
git init
echo "first content" >> first.out ; git add first.out ; git commit -m "first commit"
echo "second content" >> second.out ; git add second.out ; git commit -m "second commit"
echo "third content" >> third.out ; git add third.out ; git commit -m "third commit"
git checkout -b thirdAndFourth
echo "third content by topic branch" > third.out ; git add third.out ; git commit -m "third commit by topic branch "
git checkout main
echo "third content by main branch" > third.out ; git add third.out ; git commit -m "third commit by main branch "
git checkout thirdAndFourth 
echo "fourth content" > fourth.out ; git add fourth.out ; git commit -m "fourth commit "
git checkout main

git log main
git log thirdAndFourth 
```

2. Now merge thirdAndFourth into main:
```shell
 git merge thirdAndFourth 
 git status
```
  a. solve conflict using git checkout --theirs / --ours
  ```shell
   cat third.out
   git checkout --theirs third.out
   cat third.out
   git add third.out 
   git commit
   git log
      
  ```
  b. solve conflict using git mergetool(first go back to state before merge):
  ```shell
  git reset --hard HEAD@{1}
  git merge thirdAndFourth 
  git status
  cat third.out
  git mergetool
  cat third.out
  git status
  git clean -f third.out.orig
  git commit 
  git log
    
  ```
3. Improve log history - In both cases, regardless of how you settled the conflict, the log history is somehow confusing, as it's showing both commits of    the 2 branches in chronological order , and a merge commit, which the reader of log might miss from this log, that we favored thirdAndFourth branch'      version of file third.out when resolved the conflict, even if you edit the merge commit to reflect that, it's still a bit confusing.
   Instead of regular merge, use --sqaush option to commit yourself the commit message in a way that will reflect more what happened when we resolved 
   conflicts:
   ```shell
   git reset --hard HEAD@{1}
   git merge --squash thirdAndFourth 
   git checkout --theirs third.out 
   git add third.out 
   git status
   git commit -sm "merge thirdAndFourth branch by favoring its version of third.out, and add fourth.out"
   git log
    
   ```
4. Imporve further log history by doing git rebase with conflicts:
```shell
 git reset --hard HEAD@{1}
 git rebase main thirdAndFourth 
 git checkout --theirs third.out
 git add third.out
 ## input commit message " third commit by topic branch - favoring thirdAndFourth' version of file third.out"
 git rebase --continue
 git status
 git checkout main 
 git merge thirdAndFourth --ff-only 

[zgrinber@zgrinber git-demos]$ git log
commit af4e0852faf7c87d6cf090e68b655d4a8d286930 (HEAD -> main, thirdAndFourth)
Author: Zvi Grinberg <zgrinber@redhat.com>
Date:   Sun Aug 7 14:50:29 2022 +0300

    fourth commit

commit 22c37bd1c0ce24400da90597b4aef8f09ee2066c
Author: Zvi Grinberg <zgrinber@redhat.com>
Date:   Sun Aug 7 14:46:21 2022 +0300

    third commit by topic branch - favoring thirdAndFourth' version of file third.out

commit c6a4aa1a5e3e5904ae4bb27a2668a29fcd378e36
Author: Zvi Grinberg <zgrinber@redhat.com>
Date:   Sun Aug 7 14:46:55 2022 +0300

    third commit by main branch

commit 8503cfd166da90b5950c247015881884e546b2a9
Author: Zvi Grinberg <zgrinber@redhat.com>
Date:   Sun Aug 7 14:44:55 2022 +0300

    third commit

commit ee75eaf65fa2d78bcab5d65037a9991c012e1e26
Author: Zvi Grinberg <zgrinber@redhat.com>
Date:   Sun Aug 7 14:44:05 2022 +0300

    second commit

commit 488882a5501e7d0ce8dcd924fbda0bf782f7be84
Author: Zvi Grinberg <zgrinber@redhat.com>
Date:   Sun Aug 7 14:43:11 2022 +0300

    first commit

```
Now history looks linear and everything is clear from what happened in rebasing topic branch on top of main, better doesn't it?



