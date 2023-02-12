.. _grundlagen:

###########
Grundlagen
###########

Metasploit Module
*******************

MSP hat mehrere Module, u.a. 

1. Exploits
   
   Exploits sind Programme, die die Schwachstellen am Zielsystem ausnutzen. Im MSP Framework kann man
   nach den Schwachstellen suchen, erhält erweiterte Informationen etc. 

2. Payloads
   
   Paylods sind Aktivitäten die nach dem Exploit ausgeführt werden, z.B.:

   * Start einer reverse Shell auf dem Zielsystem
   
   * Start einer bind Shell, also einer Shell die über einen Port auf dem Zielsystem erreicht werden kann. 
   
   * Start eines meterpreter
   
   usw.

   Eine reverse Shell hat den Vorteil, dass i.d.R. der ausgehende Kommunikationsverkehr nicht durch eine
   Firewall blockiert wird. 

3. Auxiliaries

   Sind Unterstützungsprogramme in Metasploit, z.B. Sniffer, Port Scanners. Sie können aufgerufen werden, um
   mehr Informationen vom Zielsystem zu erlangen.

4. Encoders

   In Metasploit können Programme auch verschlüsselt werden, die bei Ausführung dann entschlüsselt werden. Ziel
   ist es, dass AV Programme diese nicht als Schadcode identifizieren. Da AV Programme bereits Signaturen 
   für bekannte Verschlüsselungen haben, muss man dies manuell ändern, um die Wahrscheinlichkeit zu erhöhen,
   dass AV Programme dies nicht als Schadcode identifizieren


Programme im Metasploit
************************
Metasploit ist in Ruby geschrieben und kann durch plugins erweitert werden. 

1. mfsconsole
   CLI für das Metasploit Framework. 

2. msfdb
   Eine postgres - Datenbank, die z.B. Ergebnisse eines Scans zwischenspeichern kann. 

3. msfvenom
   Tool um eigene Payloads zu erstellen und auf dem Zielsystem auszuführen. 

4. meterpreter
   ist ein erweiterter Payload mit einer mehreren Funktionalitäten. Es kann verschlüsselt 
   kommunizieren, schwer zu identifizieren. Es können Screenshoots erstellt werden, Passwörter
   extrahiert werden usw. 

 

Zurück zu :ref:`grundlagen`

