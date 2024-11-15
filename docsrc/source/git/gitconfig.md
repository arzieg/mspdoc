# Git


## Configuration

### Mehrere Git - Accounts unter VSC (https://stackoverflow.com/questions/62625513/i-have-2-github-accounts-how-can-i-use-both-when-i-am-working-in-vs-code)

1. Define some aliases in ~/.ssh/config

```  
  Host *github*
    HostName github.com
    User git
  Host perso-github.com
    IdentityFile ~/.ssh/id_rsa_perso
  Host pro-github.com
    IdentityFile ~/.ssh/id_rsa_pro
```

2. Make sure you ssh keys are added in your github accounts:

* Personal Account (git_perso_account) : ~/.ssh/id_rsa_perso.pub
* Professional Account (git_pro_account) : ~/.ssh/id_rsa_pro.pub

3. Clone your repositories as follows
   
```
  ~ # cd repositories
  ➜ repositories # git clone perso-github.com:git_perso_account/some_fun_repo.git 
  ➜ repositories # git clone pro-github.com:git_pro_account/boring_business_repo.git 
```

4. Remove git global user config


```
  ➜ repositories # git config --global --unset user.name
  ➜ repositories # git config --global --unset user.email
```

5. Add your github account informations in each repository

```
  ➜ repositories # cd some_fun_repo 
  ➜ some_fun_repo git:(master) # git config --local user.name git_perso_account
  ➜ some_fun_repo git:(master) # git config --local user.email perso@gmail.com
  ➜ some_fun_repo git:(master) # cd ../boring_business_repo
  ➜ boring_business_repo git:(master) # git config --local user.name git_pro_account
  ➜ boring_business_repo git:(master) # git config --local user.email you@business.com
```

6. fork einbinden

Möchte man einen Fork mit einbinden, muss man den Upstream konfigurieren. 

git remote -v (Anzeige aktuelle Konfiguration)
git remote add upstream perso-github.com:octocat/Spoon-Knife.git 

## Windows/Linux Repo in WSL

Wenn man das Repository unter einen Windows-Pfad hat aber über die WSL darauf zugreift, werden gerne die UNIX CR Endungen in Windows Endungen modifiziert. Das ist bei Shell-Scripten problematisch. 

daher kann man das Dateispezifisch festlegen in einer .gitattributes Datei. Das hat den Vorteil, dass beim auschecken, alle das selbe nutzen. 

```
# Set the default behavior, in case people don't have core.autocrlf set.
* text=auto

# Explicitly declare text files you want to always be normalized and converted
# to native line endings on checkout.
*.c text
*.h text

# Declare files that will always have LF line endings on checkout.
*.sh text eol=lf
*.bash text eol=lf
*.py text eol=lf

# Denote all files that are truly binary and should not be modified.
*.png binary
*.jpg binary
```

```
git config --global core.autocrlf false    (This setting ensures that Git does not automatically convert line endings.)
git config --global core.eol lf  (ensure that files are checked out with Unix line ending)
```


## Filenamen 

In Windows wird nicht zwischen groß/Kleinschreibung unterschieden. Insofern erscheint in git keine Änderung. Wenn man das möchte, muss man das explizit per
git command erzwingen: 

`git mv -f name.java Name.java`




## Cloning

Clonen einer lokalen git-Repostruktur in ein ausgechecktes Repo
  git clone file://<full path to gitrepostruktur>.git /tmp/<myrepo>