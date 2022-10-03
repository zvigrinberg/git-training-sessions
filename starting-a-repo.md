# Several Scenarios to Start a Repo

## First case: You don't have files in Local repo , and remote repo exists(Easiest):

```shell
git clone repo_address_in_SSH_or_in_http
```

## Second case: you already opened a repository on server(github/Bitbucket etc.) , have nothing special there(can wiped out) , but have a lot of files locally that you need to upload:

```shell
cd /go/to/directory/with/all/files
git init
git add -A
git commit -m "initial commit containing mass of files"
git remote add origin git@github.com:zvigrinberg/git-starting-repo.git
git push -f -u origin main 
```

## Third case: you already opened a repository on server, have important things there, but also have important things locally in a directory

Note: of course you can clone the repo locally, and then copy the files to the local worktree in directory, add to index and commit, but this is somewhat cumbersome, then let's see a way of how to do it with pure git:

```shell
cd /go/to/directory/with/all/files
git init
git add -A
git commit -m "initial commit containing important files locally"
git log
git pull --rebase  git@github.com:zvigrinberg/git-starting-repo.git
git remote add origin git@github.com:zvigrinberg/git-starting-repo.git
git push -u origin main
```

### Another way
```shell
cd /go/to/directory/with/all/files
git init
git remote add origin git@github.com:zvigrinberg/git-starting-repo.git
git pull origin main
git status
git add -A
git commit -m "initial commit containing another important files locally"
git fetch --all


git branch --set-upstream-to=origin/main main
git push

or

git push -u origin main
```


