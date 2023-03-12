.. _packages:

##########
Packages 
##########


Projektstruktur
================
app/<packagename>
    src/
       __init__.py
       <package.py>
       <other.py>
    test/
    __init__.py  (hier steht drin: from .src.<package> import (...) und für ... alle Klassen von dem Pakete <package> )
__init__.py
Readme.md 
LICENSE.txt
setup.py

setup
======
pip install wheel  -> zum erstellen der Binaries
pip install setuptools
pip install twine   -> zum publizieren

setup.py:
  file konfigurieren
    classifiers setzen
    install_requirements

build
======
python setup.py bdist_wheel  -> create binary distribution file
python setup.py sdist        -> create source destination file

pip install . (installiert es dann lokal, zum Testen)

publizieren
===========
pypi.org -> user account anlegen
test.pypi.org  -> test bevor es in das produktive repo hochgeladen wird

twine check dist/*   -> check ob alles für eine Publizierung vorhanden install
twine upload dist/*  -> Upload to pypi
twine upload -r testpypi dist/*   -> upload to testrepository


- Modules, Abstraction, Data Teil des Packages oder nicht
- Add supporting material
- richtige software lizenz verwendet
- making update easily (bumpversion z.B. als Unterstützungstools)
- use project classifiers (eine Liste ist auf pypi)



