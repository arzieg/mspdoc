# MSPDOC

Mitschriften zum Thema MSP
---------------------------

Die Mitschrift ist als Sphinx Dokumentation aufgesetzt. Zusätzlich wird github genutzt für 
das Hosting der mit Sphinx erstellten statischen Webseiten. 

Ein HowTo-Setup eines solchen Repositories ist beschrieben bei
https://www.docslikecode.com/articles/github-pages-python-sphinx/

Die Dokumentation ist erreichbar unter https://arzieg.github.io/mspdoc/index.html

1. Erstelle ein Verzeichnis docs im master/main-Branch
2. Auf github, wechsel zum Repository, Einstellungen -> Pages -> Branch (main) -> docs -> Save
3. Erstelle eine .nojekyll - Datei im Verzeichnis docs
4. Erstelle ein Verzeichnis <repo>/docsrc, hier werden die Sphinx-Sourcen abgelegt
5. Sphinx Grundstruktur erstellen (sphinx-quickstart docsrc)
6. Makefile erweitern (im Verzeichnis docsrc): 
   `github:
	    @make html
	    @cp -a build/html/. ../docs`
7. Um lokal die Dokumentation erstellen, dann `make html` eingeben.
8. Wenn die Dokumentation auf github erzeugt werden soll, dann `make github` und die Änderungen pushen




