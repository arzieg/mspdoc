.. _openqa_basic:

###############
openQA Allgemein
###############

Dir Structure
--------------

Basis Glossar: https://open.qa/docs/#concepts

/etc/openqa   -> Konfigurationsdateien für openQA

/var/lib/openqa
    share/tests/<distribution>/ 
      * Testcode je Distribution
        * lib   - libaries
        * tests - test Module
        * schedule - Testplan
        * products - Ablageort der needles
    
    factory
      * Asstes für den Aufbau eines Tests, z.B. Image, ISO, repository ...
  
    testresults
      * logfiles

Zyklisch werden die openQA Verzeichnisse aufgeräumt. Soll etwas nicht gelöscht werden, dann liegt das in einem Unterverzeichnis "fixed". Bsp.: Ablage eines 
qemu-Images unter /var/lib/openqa/factory/hdd/fixed.
Die Aufräumaktivitäten basieren auf Jobgruppen, auf der Ebene kann man ein Regelwerk definieren, z.B. wie lange z.B. Joblogs aufbewahrt werden sollen.


Einen Test schreiben
---------------------

https://open.qa/docs/#_introduction_3


1. Erstelle eine Maschine, eine Maschine hat eine Parameterisierung 
2. Erstelle ein Medium Typ (eine Distribution, eine Version, ein Flavor, eine Architektur, weitere Parameter)
3. Definiere eine Testkollektion von Tests die laufen sollen im "Test suites" Menü
4. Definiere Job Gruppen (hier laufen die Informationen aus 1-3 zusammen, diese müssen im Template angegeben werden) <https://open.qa/docs/#_configuring_job_groups_via_yaml_documents>
5. Starte Job Gruppen für die Ausführung von Tests

"Machines, mediums, test suites and job templates can all set various configuration variables. The so called job templates within the job groups define 
how the test suites, mediums and machines should be combined in various ways to produce individual 'jobs'." <https://open.qa/docs/#_introduction_3>



