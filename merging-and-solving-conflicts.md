
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
[zgrinber@zgrinber git-demos]$ git init
Initialized empty Git repository in /home/zgrinber/git/git-demos/.git/
[zgrinber@zgrinber git-demos]$ echo "first content" >> first.out ; git add first.out ; git commit -m "first commit"
[main (root-commit) 488882a] first commit
 1 file changed, 1 insertion(+)
 create mode 100644 first.out

[zgrinber@zgrinber git-demos]$ echo "second content" >> second.out ; git add second.out ; git commit -m "second commit"
[main ee75eaf] second commit
 1 file changed, 2 insertions(+)
 create mode 100644 second.out
[zgrinber@zgrinber git-demos]$ echo "third content" >> third.out ; git add third.out ; git commit -m "third commit"
[main 8503cfd] third commit
 1 file changed, 1 insertion(+)
 create mode 100644 third.out
[zgrinber@zgrinber git-demos]$ git checkout -b thirdAndFourth
Switched to a new branch 'thirdAndFourth'
[zgrinber@zgrinber git-demos]$ echo "third content by topic branch" > third.out ; git add third.out ; git commit -m "third commit by topic branch "
[thirdAndFourth e240c26] third commit by topic branch
 1 file changed, 1 insertion(+), 1 deletion(-)
[zgrinber@zgrinber git-demos]$ git checkout main
Switched to branch 'main'

[zgrinber@zgrinber git-demos]$ echo "third content by main branch" > third.out ; git add third.out ; git commit -m "third commit by main branch "
[main c6a4aa1] third commit by main branch
 1 file changed, 1 insertion(+), 1 deletion(-)
 
[zgrinber@zgrinber git-demos]$ git checkout thirdAndFourth 
Switched to branch 'thirdAndFourth'
[zgrinber@zgrinber git-demos]$ echo "fourth content" > fourth.out ; git add fourth.out ; git commit -m "fourth commit "
[thirdAndFourth 534ec35] fourth commit
 1 file changed, 1 insertion(+)
 create mode 100644 fourth.out
[zgrinber@zgrinber git-demos]$ git checkout main
Switched to branch 'main'

[zgrinber@zgrinber git-demos]$ git log main
commit c6a4aa1a5e3e5904ae4bb27a2668a29fcd378e36 (HEAD -> main)
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
    
[zgrinber@zgrinber git-demos]$ git log thirdAndFourth 
commit 534ec356a10f0f3e5ee2b7a5648f4220e45d8e51 (thirdAndFourth)
Author: Zvi Grinberg <zgrinber@redhat.com>
Date:   Sun Aug 7 14:50:29 2022 +0300

    fourth commit

commit e240c26572fa1ac6f5df9c727b9f9c39bf350c22
Author: Zvi Grinberg <zgrinber@redhat.com>
Date:   Sun Aug 7 14:46:21 2022 +0300

    third commit by topic branch

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

2. Now merge thirdAndFourth into main:
```shell
[zgrinber@zgrinber git-demos]$ git merge thirdAndFourth 
Auto-merging third.out
CONFLICT (content): Merge conflict in third.out
Automatic merge failed; fix conflicts and then commit the result.
[zgrinber@zgrinber git-demos]$ git status
On branch main
You have unmerged paths.
  (fix conflicts and run "git commit")
  (use "git merge --abort" to abort the merge)

Changes to be committed:
	new file:   fourth.out

Unmerged paths:
  (use "git add <file>..." to mark resolution)
	both modified:   third.out
```
  a. solve conflict using git checkout --theirs / --ours
  ```shell
  [zgrinber@zgrinber git-demos]$ cat third.out
    <<<<<<< HEAD
    third content by main branch
    =======
    third content by topic branch
    >>>>>>> thirdAndFourth
    [zgrinber@zgrinber git-demos]$ git checkout --theirs third.out
    Updated 1 path from the index
    [zgrinber@zgrinber git-demos]$ cat third.out
    third content by topic branch
    [zgrinber@zgrinber git-demos]$ git add third.out 
    [zgrinber@zgrinber git-demos]$ git commit
    [main c38fa1c] Merge branch 'thirdAndFourth'
    
    [zgrinber@zgrinber git-demos]$ git log
    commit c38fa1c59f2a4d66958c00511278df2f1e4de836 (HEAD -> main)
    Merge: c6a4aa1 534ec35
    Author: Zvi Grinberg <zgrinber@redhat.com>
    Date:   Sun Aug 7 15:03:56 2022 +0300

        Merge branch 'thirdAndFourth'

    commit 534ec356a10f0f3e5ee2b7a5648f4220e45d8e51 (thirdAndFourth)
    Author: Zvi Grinberg <zgrinber@redhat.com>
    Date:   Sun Aug 7 14:50:29 2022 +0300

        fourth commit

    commit c6a4aa1a5e3e5904ae4bb27a2668a29fcd378e36
    Author: Zvi Grinberg <zgrinber@redhat.com>
    Date:   Sun Aug 7 14:46:55 2022 +0300

        third commit by main branch

    commit e240c26572fa1ac6f5df9c727b9f9c39bf350c22
    Author: Zvi Grinberg <zgrinber@redhat.com>
    Date:   Sun Aug 7 14:46:21 2022 +0300

        third commit by topic branch

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
  b. solve conflict using git mergetool 
  ```shell
  [zgrinber@zgrinber git-demos]$ cat third.out
  <<<<<<< HEAD
  third content by main branch
  =======
  third content by topic branch
  >>>>>>> thirdAndFourth
  [zgrinber@zgrinber git-demos]$ git mergetool

  This message is displayed because 'merge.tool' is not configured.
  See 'git mergetool --tool-help' or 'git help config' for more details.
  'git mergetool' will now attempt to use one of the following tools:
  meld opendiff kdiff3 tkdiff xxdiff tortoisemerge gvimdiff diffuse diffmerge ecmerge p4merge araxis bc codecompare smerge emerge vimdiff nvimdiff
  Merging:
  third.out

  Normal merge conflict for 'third.out':
    {local}: modified file
    {remote}: modified file
  Hit return to start merge resolution tool (meld): 
  [zgrinber@zgrinber git-demos]$ cat third.out
  third content by topic branch

  [zgrinber@zgrinber git-demos]$ git add .
  [zgrinber@zgrinber git-demos]$ git commit 
  [main 9c6046e] Merge branch 'thirdAndFourth'
  [zgrinber@zgrinber git-demos]$ git log
  commit 9c6046eacc8c4a938c51de014a5164c9632aff86 (HEAD -> main)
  Merge: c6a4aa1 534ec35
  Author: Zvi Grinberg <zgrinber@redhat.com>
  Date:   Sun Aug 7 15:11:02 2022 +0300

      Merge branch 'thirdAndFourth'

  commit 534ec356a10f0f3e5ee2b7a5648f4220e45d8e51 (thirdAndFourth)
  Author: Zvi Grinberg <zgrinber@redhat.com>
  Date:   Sun Aug 7 14:50:29 2022 +0300

      fourth commit

  commit c6a4aa1a5e3e5904ae4bb27a2668a29fcd378e36
  Author: Zvi Grinberg <zgrinber@redhat.com>
  Date:   Sun Aug 7 14:46:55 2022 +0300

      third commit by main branch

  commit e240c26572fa1ac6f5df9c727b9f9c39bf350c22
  Author: Zvi Grinberg <zgrinber@redhat.com>
  Date:   Sun Aug 7 14:46:21 2022 +0300

      third commit by topic branch

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
3. Improve log history - In both cases, regardless of how you settled the conflict, the log history is somehow confusing, as it's showing both commits of    the 2 branches in chronological order , and a merge commit, which the reader of log might miss from this log, that we favored thirdAndFourth branch'      version of file third.out when resolved the conflict, even if you edit the merge commit to reflect that, it's still a bit confusing.
   Instead of regular merge, use --sqaush option to commit yourself the commit message in a way that will reflect more what happened when we resolved 
   conflicts:
   ```shell
   [zgrinber@zgrinber git-demos]$ git reset --hard <commit-id-before-previous-merge-from-reflog>
   [zgrinber@zgrinber git-demos]$ git merge --squash thirdAndFourth 
    Auto-merging third.out
    CONFLICT (content): Merge conflict in third.out
    Squash commit -- not updating HEAD
    Automatic merge failed; fix conflicts and then commit the result.
    [zgrinber@zgrinber git-demos]$ git checkout --theirs third.out 
    Updated 1 path from the index
    [zgrinber@zgrinber git-demos]$ git add third.out 
    [zgrinber@zgrinber git-demos]$ git status
    On branch main
    Changes to be committed:
      (use "git restore --staged <file>..." to unstage)
      new file:   fourth.out
      modified:   third.out

    [zgrinber@zgrinber git-demos]$ git commit -sm "merge thirdAndFourth branch by favoring its version of third.out, and add fourth.out"
    [main 4225382] merge thirdAndFourth branch by favoring its version of third.out
     2 files changed, 2 insertions(+), 1 deletion(-)
     create mode 100644 fourth.out
    [zgrinber@zgrinber git-demos]$ git log
    commit 4225382120d450dfcfd9abd2c00cbb5530ba4904 (HEAD -> main)
    Author: Zvi Grinberg <zgrinber@redhat.com>
    Date:   Sun Aug 7 15:30:28 2022 +0300

        merge thirdAndFourth branch by favoring its version of third.out

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
4. Imporve further log history by doing git rebase with conflicts:
```shell
[zgrinber@zgrinber git-demos]$ git rebase main thirdAndFourth 
Auto-merging third.out
CONFLICT (content): Merge conflict in third.out
error: could not apply e240c26... third commit by topic branch
Resolve all conflicts manually, mark them as resolved with
"git add/rm <conflicted_files>", then run "git rebase --continue".
You can instead skip this commit: run "git rebase --skip".
To abort and get back to the state before "git rebase", run "git rebase --abort".
Could not apply e240c26... third commit by topic branch
[zgrinber@zgrinber git-demos]$ git checkout --theirs third.out
Updated 1 path from the index
[zgrinber@zgrinber git-demos]$ git add third.out
[zgrinber@zgrinber git-demos]$ git rebase --continue
[detached HEAD 22c37bd] third commit by topic branch - favoring thirdAndFourth' version of file third.out
[zgrinber@zgrinber git-demos]$ git status
On branch thirdAndFourth
nothing to commit, working tree clean
[zgrinber@zgrinber git-demos]$ git checkout main 
Switched to branch 'main'
[zgrinber@zgrinber git-demos]$ git merge thirdAndFourth --ff-only 
Updating c6a4aa1..af4e085
Fast-forward
 fourth.out | 1 +
 third.out  | 2 +-
 2 files changed, 2 insertions(+), 1 deletion(-)
 create mode 100644 fourth.out
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



