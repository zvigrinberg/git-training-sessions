# git reflog for advanced recovery, git log for showing commit tree and basic recovery
## Agenda
1. See malicious/on purpose deleting files commit and its content.
2. see how to recover/restore commit and content using git log
3. See malicious on purpose deleting files and history of a main branch
4. See how to restore/recover history + content using git reflog. 


## Procedure
```shell
   git clone /tmp/git-recovery/
   git log
   cd git-recovery
   ll
   
   ##case 1 - basic recovery.
   git log
   ## on purpose deleting files from branch, commit them, and push to remote server.
   git rm -rf *
   git commit -m "delete all files"
   ll
   git push
   ## reposition and setting HEAD to previous commit according to git log
   git reset --hard 1a40853d5af4a561405da51546581c30c009590d
   git log
   ll
   ## force push to remote server to recover lost files..
   git push -f
   
   ##case 2 - advanced recovery.
   git checkout --orphan=evil
   ll
   git log
   git rm -rf *
   git commit -m "delete history and all content" --allow-empty
   git log
   git push -f -u origin evil:main
   git branch -D main
   git branch -av
   ll
   git log
   git reflog
   git checkout HEAD@{3} -b main
   ll
   git log
   git push -f -u origin main
   ```
