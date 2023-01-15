.. _filestruktur:

Filestruktur
###############

Metasploit wird installiert nach /usr/share/metasploit-framework

ls -l /usr/share/metasploit-framework/modules

drwxr-xr-x 22 root root 4096 Dec  5 08:38 auxiliary    <- Untersützungsprogramme
drwxr-xr-x 12 root root 4096 Dec  5 08:38 encoders     <- Verschlüsselungsprogramme
drwxr-xr-x  3 root root 4096 Dec  5 08:38 evasion      <- helper zur Verschleierung der Paylods vor AV Programmen
drwxr-xr-x 22 root root 4096 Dec  5 08:38 exploits     <- Exploits zur Ausnutzung einer Schwachstelle
drwxr-xr-x 12 root root 4096 Dec  5 08:38 nops         <- no operation
drwxr-xr-x  6 root root 4096 Dec  5 08:38 payloads     <- Programme die auf dem Zielsystem nach dem Exploit abgesetzt werden
drwxr-xr-x 14 root root 4096 Dec  5 08:38 post         <- post Programme 




Zurück zu :ref:`filestruktur`