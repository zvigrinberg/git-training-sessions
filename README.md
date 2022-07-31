# git-training-sessions
## The following git commands are in the scope of git advanced sessions:

Intro: Give brief on index, working-tree(working area), local-repo, remote repo.

git remote

git reset --hard --soft --mixed

git rm -rf --cached

git add [--intent-to-add] [-A] [-u - only tracked files]

git fetch [--prune] , [--all] , fetch remote/origin(update remote refs) 

git pull -- dry-run , --rebase

git stash save, apply, pop,clear,show

git rebase -i

git restore

git revert

divide and remain here only basic checkout flags:

git checkout specific file, -p(hunks) , ref + specific file, checkout to resolve, 
 conflicts with ours/theirs, --orpahn , to start a branch with a clean history, checkout -b to start new branch , git checkout HEAD . --no-overlay eliminate all indexed and not staged files from index and working tree(like git reset --hard), checkout from rev-list, checkout from reflog/log , and etc.

git push -u origin new-branch-name(along with git checkout -b new-branch-name).



git commit --amend --no-edit --signoff by user.
git log branch_name/tag name

git diff HEAD, --cached(only comparing staged files with reference to HEAD/Commit ID.

git diff COMMIT_ID/REF/tag.

git diff REF1, REF2.

git merge --squash branch/ref/commit.

git merge git merge -s recursive -Xtheirs / -Xours / <allow-unrelated-histories>

git show commit-id [--name-only] [--format=email/fuller]

git blame -  git blame hello.there -l(long revisions)


git merge -s octopus HEAD@{1} HEAD@{6} / branchA BranchB BranchC...\


git cherry-pick

git rev-list --all --count
 
 git submodule(optional)
 
git reflog
 
(*)git request-pull -p start-commit, repo/url, end commit
 
 ## git bisect demo
 Link to repo that show how to use git bisect with example [can be found here](https://github.com/zvigrinberg/git-bisect-demo)
 
 git bisect
