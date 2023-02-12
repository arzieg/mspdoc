.. _filestruktur:

Filestruktur
###############

Metasploit wird installiert nach /usr/share/metasploit-framework

.. code:: bash

    ls -l /usr/share/metasploit-framework/modules

    auxiliary    <- Untersützungsprogramme
    encoders     <- Verschlüsselungsprogramme
    evasion      <- helper zur Verschleierung der Paylods vor AV Programmen
    exploits     <- Exploits zur Ausnutzung einer Schwachstelle
    nops         <- no operation
    payloads     <- Programme die auf dem Zielsystem nach dem Exploit abgesetzt werden
    post         <- post Programme 




Zurück zu :ref:`filestruktur`