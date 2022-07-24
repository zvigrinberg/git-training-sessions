# git-training-sessions
## The following git commands are in the scope of git advanced sessions:

git reset --hard --soft --mixed

git reflog

git rebase -i

git checkout specific file, -p(hunks) , ref + specific file, checkout to resolve, 
 conflicts with ours/theirs, --orpahn , to start a branch with a clean history, checkout -b to start new branch , git checkout HEAD . --no-overlay eliminate all indexed and not staged files from index and working tree(like git reset --hard), checkout from rev-list, checkout from reflog/log , and etc.

git push -u origin new-branch-name(along with git checkout -b new-branch-name).

git fetch [--prune] , [--all] , fetch remote/origin(update remote refs) 

git pull -- dry-run , --rebase

git merge git merge -s recursive -Xtheirs / -Xours / <allow-unrelated-histories>

git merge -s octopus HEAD@{1} HEAD@{6} / branchA BranchB BranchC...\
git merge --squash branch/ref/commit.

git commit --amend --no-edit

git log branch_name/tag name

git rm -rf --cached

git add [--intent-to-add] [-A] [-u - only tracked files]

git stash save, apply, pop,clear,show

git diff HEAD, --cached(only comparing staged files with reference to HEAD/Commit ID.

git diff COMMIT_ID/REF/tag.

git diff REF1, REF2.

git request-pull -p start-commit, repo/url, end commit

git show commit-id [--name-only] [--format=email/fuller]

git blame -  git blame hello.there -l(long revisions)

git bisect

git restore

git cherry-pick

git remote

git rev-list --all --count
 
 git submodule
 
