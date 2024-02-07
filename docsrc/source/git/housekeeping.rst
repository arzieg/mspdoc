.. _git_housekeeping:

#################
Git Housekeeping
#################

https://nickymeuleman.netlify.app/blog/delete-git-branches

git branch --merged             List all local merged branches.           

git branch --no-merged          List all local unmerged branches.         

git branch -r --merged          List all remote merged branches.          

git branch -r --no-merged       List all remote unmerged branches.        

git branch -d <branchname>      lokal löschen

git branch -D <branch>          lokalen unmerged Branch löschen!!

git push origin --delete <branch> remote löschen

lokale branches aufräumen, die remote gelöscht sind:
git remote prune origin --dry-run  (ohne dry-run wird es umgesetzt)
oder git fetch --prune
