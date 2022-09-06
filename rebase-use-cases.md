# Rebase use-cases 

## When to use - Use-Cases
1. When you want to integrate changes of feature/topic branch with mainstream branch and you want a linear history that left no clues or evidences of 
   merges, and merging the branch to mainstream branch will result in a merge commit because fast-forwarding the HEAD is not an option(only recursive 3  
   ways merge is possible)
   
2. When you started a feature branch featureA, and started to work on it, and then you forked another feature branch from it - featureB, and continue to work on it also, in this case, you might need to use rebase in the following 3 cases:

      a. main branch progressed and featureB needs these updates in order to progress and for testing.   
     
      b. if you did it by mistake and featureB already done work with at least 1 or 2 commits, and featureB has nothing to do with featureA, and 
         don't want to be dependent on it(if featureA deleted without being merged to main then featureB remains/stuck with irrelevant commits and work).
         
      c. if they're related, but only featureB is ready and can be integrated into main, but featureA still needs additional work.
   
      
3. When you created a feature branch, and created a series of commits in it, and then you realize at the end, just before openning a PR to main, that you missed some work in the middle , and you want the history to be shown like this commit with its work is in the middle, although you "inserted" it in the past in retrospective with rebasing.

4. When you created a feature branch, and created a lot of commits , and some of them you don't want to be merged into main branch at all, some of them you want to squash into another, and some of them you want to change commit message, the rebase option that lets you do all of that in one shot is git rebase -i(re-writing history)

## When not to use

 1. Do not use rebasing to replay commmits that already pushed to remote repo that other colleagues already pulled and worked with.
    in other words, if main is the mainstream branch, and feature is some feature forked from main and is local, then use - rebase main feature, because rebase is changing commit id hash values, and otherwise it can be very confusing


### Procedures
#### Use case #1
```shell
git checkout main
for i in {1..4} ; do echo "sample work $i" >> $i.out; git add $i.out; git commit -m "sample work $i"; done
git log
git checkout -b feature/additionalWork
for i in {5..8} ; do echo "additional work $i" >> $i.out; git add $i.out; git commit -m "additional work $i"; done
git checkout main
echo "create directly commit in main" >> main.out; git add main.out; git commit -m "create directly commit in main"
git checkout feature/additionalWork 
git rebase main
git log
git checkout main
git merge feature/additionalWork
git log
```

#### Use case #2
```shell
git checkout -b feature2
for i in {9..15} ; do echo "sample work $i" >> $i.out; git add $i.out; git commit -m "sample work $i"; done
git checkout -b feature3
for i in {16..22} ; do echo "additional work $i" >> $i.out; git add $i.out; git commit -m "additional work $i"; done
git log
git checkout main -b feature/important
for i in {23..24} ; do echo "infra work $i" >> $i.out; git add $i.out; git commit -m "infrastructure work $i"; done
git checkout main
git merge feature/important
git branch -d feature/important
log
git checkout feature3
git rebase --onto=main feature2 feature3
git branch -D feature2
git checkout main
git merge feature3
git log

git merge feature3
git branch -d feature3
git log
```

#### Use case #3
```shell
   
   git checkout -b feature/something
   for i in {25..31} ; do echo "sample number $i" >> $i.out; git add $i.out; git commit -m "commit number $i"; done
   git log
   git checkout HEAD@{4}
   git show HEAD
   git tag newBase
   git checkout feature/something
   git log
   git checkout -b temp newBase
   git log
   echo "sample number 27-28" >> 2728.out ; git add 2728.out ; git commit -m "commit number after 27, before 28"
   git log
   git rebase temp feature/something
   git status
   git log
   git checkout main
   git merge feature/something
   git log
   git branch -d feature/something temp
   git tag -d newBase
   git log

```

#### Use case #4 - Rebase Interactive(git rebase -i):
[Link to repo with demo](https://github.com/zvigrinberg/git-interactive-rebase-demo/)




