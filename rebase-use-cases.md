# Rebase use-cases 

## When to use - Use-Cases
1. When you want to integrate changes of feature/topic branch with mainstream branch and you want a linear history that left no clues or evidences of 
   merges, and merging the branch to mainstream branch will result in a merge commit because fast-forwarding the HEAD is not an option(only recursive 3  
   ways commit)
   
2. When you started a feature branch featureA, and started to work on it, and then you forked another feature branch from it - featureB, and continue to work on it also, in this case, you might need to use rebase in the following 3 cases:

      a. main branch progressed and featureB needs these updates in order to progress and for testing.   
     
      b. if you did it by mistake and featureB already done work with at least 1 or 2 commits, and featureB has nothing to do with featureA, and 
         don't want to be dependant on it(if featureA deleted without being merged to main then featureB remains/stuck with irrelevant commits and work).
         
      c. if they're related, but only featureB is ready and can be integrated into main, but featureA still needs additional work.
   
      
3. When you created a feature branch, and created a series of commits in it, and then you realize at the end, just before openning a PR to main, that you missed some work in the middle , and you want the history to be shown like this commit with its work is in the middle, although you "inserted" it in the past in retrospective with rebasing.


## When not to use

 1. Do not use rebasing to replay commmits that already pushed to remote repo that other colleagues already pulled and worked with.
    in other words, if main is the mainstream branch, and feature is some feature forked from main,  use -  rebase main feature, because rebase is changing
    commit dates and hash values.


### Procedures

#### Use case #3
```shell
   for i in {1..6} ; do echo "sample number $i" >> $i.out; git add $i.out; git commit -m "commit number $i"; done
   git log
   git checkout -b feature
   for i in {8..14} ; do echo "sample number $i" >> $i.out; git add $i.out; git commit -m "commit number $i"; done
   git log
   git checkout 90fc40b70777ec23fe09ac1d9787cdc86d6cc403
   git tag newBase
   git checkout feature
   git log
   git checkout -b temp newBase
   git log
   echo "sample number 7" >> 7.out ; git add 7.out ; git commit -m "commit number 7"
   git log
   git rebase temp feature
   git status
   git log
   git checkout main
   git merge feature
   git log
   git branch -d feature temp
   git tag -d newBase
   git log


```
