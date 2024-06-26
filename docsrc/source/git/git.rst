.. _git:

##########
Git
##########




Speichern des Arbeitsstandes
===============================

git stash speichert den Arbeitsstand

git stash apply zurückholen des Arbeitsstandes. Der Arbeitsstand bleibt in der Liste (git stash list)

    git stash pop throws away the (topmost, by default) stash after applying it, whereas git stash apply leaves it in the stash list for possible later reuse 
    (or you can then git stash drop it). This happens unless there are conflicts after git stash pop, in which case it will not remove the stash, leaving it to 
    behave exactly like git stash apply. Another way to look at it: git stash pop is git stash apply && git stash drop.

GIT TAG
========
git tag -n          - zeige tags an inkl. message
git tag -d <tag>    - löschen lokal
git push --delete origin <tagname>
git tag eddi-lorup-r19-2 eddi-lorup-r19-2^{} -f -a    - wie ändere ich eine TAG Nachricht (solange noch nicht gepusht)


GIT History
============
git log --follow -p -- <file>      - zeige die Historie an

Secrets ausversehen gepusht
============================

Fall 1: Löschen eines Files
----------------------------

https://articles.wesionary.team/oops-pushed-secret-keys-to-github-lets-undo-that-29de9b1bddb0

1. Wipe the local mess
   Der Befehl muss vom root-Verzeichnis des repo durchgeführt werden.

.. code-block:: bash

    git filter-branch --force --index-filter "git rm --cached --ignore-unmatch path/to/secret/file" --prune-empty --tag-name-filter cat -- --all

2. Push

.. code-block:: bash

    git push origin main --force 


Fall 2: Löschen eines Secrets in einem File
---------------------------------------------

1. Modify the file locally
2. Commit der Änderungen

.. code-block:: bash

    git add path/to/your/file
    git commit -m "Removed secret key"

3. Eradicate the Secret Key from Your Commit History

.. code-block:: bash

    git filter-branch -f --tree-filter "sed -i '' 's/YOUR_SECRET_KEY_HERE/REPLACEMENT_TEXT_OR_EMPTY/g' path/to/your/file" HEAD

4. push to github

.. code-block:: bash

    git push origin main --force  # Assuming your branch is named "main"

    