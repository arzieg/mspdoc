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

daher: 

```
git config --global core.autocrlf false    (This setting ensures that Git does not automatically convert line endings.)
git config --global core.eol lf  (ensure that files are checked out with Unix line ending)
```

You can create a .gitattributes file in your repository to enforce consistent line endings. Add the following line to the file:
`* text eol=lf`




## Cloning

Clonen einer lokalen git-Repostruktur in ein ausgechecktes Repo
  git clone file://<full path to gitrepostruktur>.git /tmp/<myrepo>