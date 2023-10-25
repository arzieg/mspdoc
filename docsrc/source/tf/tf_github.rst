.. _tf_github:

##############################
Terraform Github Integration
##############################

Module entwickeln sich weiter und sollten versioniert werden in git. 

In das module-Verzeichnis wechseln

.. code-block:: shell

    cd modules
    git init
    git add .
    git commit -m "First commit"
    git remote add origin "<remote url>"
    git push origin master
    git tag -a "v0.0.1" -m "first commit"
    git push --follow-tags

In der main.tf wird nun anstelle der lokalen source für das Module das gitrepo
angegeben

.. code-block:: shell

    module "server" {
      source = "git@github.com:<company>/proj1-modules.git//services?ref=v0.0.1"
      ...
      }

Wenn man einen Pfad ansprechen möchte im git-repo (z.B. das Unterverzeichnis services), dann ist nach der URL ein //<Pfad> einzugeben.
Um die richtige Version zu verwenden, kann man den tag angeben, hier v0.0.1

Damit ist man in der Lage, z.B. in der live Umgebung unterschiedliche Modulversionen zu testen, wenn man wie hier mit git tags arbeitet.

Leider wollte Terraform leider nicht so, wie in der Dokumentation beschrieben. Zuästlich notwendig war noch 
die Konfiguration des ssh-agents

.. code-block:: shell

    vi ~/.ssh/config

    Host github.com
        HostName github.com
        user git
        IdentityFile ~/.ssh/<github key>


Module weiterentwickeln
========================

Änderungen in der Modulsyntax werden dann wie gewohnt weiterentwickelt und die Änderungen 
hochgepusht. 

1. entwickeln
2. git commit -m "Second release"
3. git push
4. git tag -a "v0.0.2" -m "Second release"
5. git push --follow-tags

und in der main.tf dann die Version anheben. So können z.B. unterschiedliche Versionen in prod und test laufen.

.. code-block:: shell

    module "server" {
      source = "git@github.com:<company>/proj1-modules.git//services?ref=v0.0.2"
      ...
      }