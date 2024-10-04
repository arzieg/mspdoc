# Git

## Speichern des Arbeitsstandes

git stash speichert den Arbeitsstand

git stash apply zurückholen des Arbeitsstandes. Der Arbeitsstand bleibt in der Liste (git stash list)

    git stash pop throws away the (topmost, by default) stash after applying it, whereas git stash apply leaves it in the stash list for possible later reuse 
    (or you can then git stash drop it). This happens unless there are conflicts after git stash pop, in which case it will not remove the stash, leaving it to 
    behave exactly like git stash apply. Another way to look at it: git stash pop is git stash apply && git stash drop.

## GIT TAG

git tag -n          - zeige tags an inkl. message
git tag -d <tag>    - löschen lokal
git push --delete origin <tagname>
git tag eddi-lorup-r19-2 eddi-lorup-r19-2^{} -f -a    - wie ändere ich eine TAG Nachricht (solange noch nicht gepusht)


## GIT History

git log --follow -p -- <file>      - zeige die Historie an

## Feature-Branch von main aktualisieren

```
git checkout b1
git merge origin/master
git push origin b1
```

alternativ

```
git fetch
git rebase origin/master
```

## Revover last commit

Branch gelöscht oder falschen commit resetet:

```
git reflog
```

## git bisect: Find the Commit That Broke Your Code

git bisect performs a binary search through your commit history to find the exact commit where things went wrong

```
git bisect start
git bisect good <last known good commit>
git bisect bad <bad commit>
```

## git cherry-pick: Selectively Apply Commits Across Branches

Sometimes you don’t want to merge an entire branch but still need to include specific commits. git cherry-pick allows you to take specific commits from one branch and apply them to another.

```
git cherry-pick <commit-hash>
```

## git reset — hard: Reset Your Code Completely

git reset — — hard is a powerful but dangerous command. It resets your working directory to a specific commit, discarding any changes that have not been committed.

```
git reset --hard <commit-hash>
```

Say your project is in a broken state and you want to start fresh from a previous commit. git reset — — hard allows you to revert all changes back to a stable commit. Just be cautious, as it removes uncommitted changes.

## git blame: Who Broke This Line of Code?

git blame shows you a detailed history of who changed what, and when.

```
git blame <file>
``` 

## git clean: Clean Up Untracked Files

Over time, your project can accumulate many untracked files, like build artifacts or logs. git clean helps you clean up by removing these files from your working directory.

```
git clean -f
git clean -fd
```

## git shortlog: Summarize Contributions by Author

```
git shortlog -s -n
```


## Secrets ausversehen gepusht


### Fall 1: Löschen eines Files


https://articles.wesionary.team/oops-pushed-secret-keys-to-github-lets-undo-that-29de9b1bddb0

1. Wipe the local mess
   Der Befehl muss vom root-Verzeichnis des repo durchgeführt werden.

```
git filter-branch --force --index-filter "git rm --cached --ignore-unmatch path/to/secret/file" --prune-empty --tag-name-filter cat -- --all
``` 

2. Push

```
git push origin main --force 
``` 

### Fall 2: Löschen eines Secrets in einem File


1. Modify the file locally
2. Commit der Änderungen

```
git add path/to/your/file
git commit -m "Removed secret key"
```

3. Eradicate the Secret Key from Your Commit History

```
git filter-branch -f --tree-filter "sed -i '' 's/YOUR_SECRET_KEY_HERE/REPLACEMENT_TEXT_OR_EMPTY/g' path/to/your/file" HEAD
``` 

4. push to github

```
git push origin main --force  # Assuming your branch is named "main"
```

Variante:

https://dev.to/kodebae/how-to-remove-a-leaked-env-file-from-github-permanently-3lei

Bsp. env-File mit secret wurde gepusht

1. Remove env file and commit 

```
bash
Copy code
git rm --cached .env
echo ".env" >> .gitignore
git add .gitignore
git commit -m "Remove .env file and add to .gitignore"
```

2. Remove the .env File from History with filter-branch

```
bash
Copy code
git filter-branch --force --index-filter 'git rm --cached --ignore-unmatch .env' --prune-empty --tag-name-filter cat -- --all
```

3. Force Push the Changes

```
bash
Copy code
git push --force --all
git push --force --tags
```

4. Clean Up Local Repository

```
bash
Copy code
rm -rf .git/refs/original/
git reflog expire --expire=now --all
git gc --prune=now --aggressive
```

5. Revoke Any Leaked Credentials

**As with any method, if your .env file contained sensitive information, revoke and regenerate those credentials immediately.**


# GIT - VSC Integration Windows 10

0. GIT installieren
1. ssh-agent aktivieren in Windows 10 als Administrator
   
   ```
   Get-Service ssh-agent | Set-Service -StartupType Automatic -PassThru | Start-Service

   nur wenn man nicht neustarten möchte:

   start-ssh-agent.cmd
   ``` 

2. Prüfen per Powershell (Userkontext)
   
   ```Powershell

   PS C:\Users\arzieg> get-service ssh-agent

    Status   Name               DisplayName
    ------   ----               -----------
    Running  ssh-agent          OpenSSH Authentication Agent
    ```

3. Anpassen .ssh/config

   ```
   Host github.com
     Hostname github.com
     User git
     IdentityFile "C:\\Users\\arzieg/.ssh/arzieg_eddi_github"
   ```

4. ssh-key laden, Powershell Usercontext

    ```Powershell
    ssh-add <Pfad zum private Key>
    ssh-add -l 
    ```

5. Test Connect 

   ```
   ssh -T git@github.com
   ``` 

5. Windows 10 hat auf ein eigenes openssh installiert. Dies funktioniert irgendwie nicht mit
   dem ssh-agent, d.h. man wird trotz agent nach einem Passwort gefragt.
   https://www.teapotcoder.com/post/how-to-fix-git-ssh-asking-for-password-on-windows-10/

   Daher am besten Environmentvariable setzen

   Windows Start -> Einstellungen -> Umgebungsvariablen für dieses Konto bearbeiten 

    Hinzufügen -> GIT_SSH_COMMAND -> C:/WINDOWS/System32/OpenSSH/ssh.exe


