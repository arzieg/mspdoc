# Pacemaker Architektur

## Key Concepts

Fencing: Aktion der Clustersoftware, um den Zugriff eines Clusterknoten auf gemeinsame Clusterressourcen yu entziehen. Fencing erfolgt durch eine Fencing-Methode oder Fencing-Device.

STONITH: "Shoot The Other Node In The Head". Ein Acronym für eine Fencing-Methode mit dem Ziel, dass ein Knoten einen anderen Knoten aus dem Clusterverbund entfernt. Die Ausführung der Fencing-Methoder läuft über ein Stonith-Agent (stonith-ng)

Ziel ist es sicherzustellen, dass bei Feststellung eines Problems eines Knotens, dieser die Clusterressourcen freigibt und ein anderer Knoten davon ausgehen kann, diese Clusterressourcen ohne Probleme verwenden zu können (bspw. Schwenk einer IP Adresse). Wenn Knoten abrupt den Cluster verlassen, wartet jeder beteiligte Cluster-Knoten, bis die Fencing-Methode beendet wurde, bevor es die Clusterressourcen der verlorenen Knotens übernimmt. 

Fencing-Gründe: 
* der Knoten ist kein guter Kandidat, um die Ressource weiterhin zu betreiben
* der Knoten hat Ressourcen in einen undefinierten (unclean) Status hinterlassen, so dass ein Reset ausgelöst wird, damit die Ressourcen wieder in einem definierten Zustand versetzt werden.
* der Kontakt zu den Cluster-Membern geht verloren.

Pacemaker ist ein Ressourcemanager. Jeder Ressource kann eine Fencing-Methode hinzugefügt werden. Verläuft eine Stop-Methode nicht erfolgreich, wird ein Fencing ausgelöst. 
(im Code: *op stop on-fail=fence* ), die Zustandseingenschaften werden korrigiert und ein anderer Knoten kann die Ressource starten.
Die Fence-Methode kann von einem anderen Knoten ausgeführt werden, es gibt aber auch Aktionen, in dem die Fence-Methode vom eigenen Knoten ausgeführt wird (self_fence). 

Quorum-Entscheidung: 
* Ein Knoten kann aufgrund einer Aktivität ggfs. nicht mehr mit den anderen Knoten kommunizieren (Last, Netzwerkausfall, ...)
* Quorum ist eine Regel, bei dem nur ein Teil des Cluster das Recht erhält, den Betrieb der Ressourcen fortzuführen (das kann z.B. die Mehrheit sein, ober aber ein Supernode entscheidet, welche Knoten weiter als "healthy" angesehen werden). 
* Ein Knoten kann nun den Status erhalten, dass es zu einem Quorum gehört oder aber nicht (sofern dieser noch darauf reagieren kann). Gehört ein Knoten nicht mehr zum Quorum, wird es aus dem Clusterkonstrukt ausgeschlossen (über eine Fencing-Methode). Erst wenn die STONITH Methode als erfolgreich abgeschlossen bestätigt wird, übernehmen die Quorum-Knoten den Service. 
* Ein Split-Brain ist eine Situation, in dem Clusterknoten untereinander nicht mehr miteinander kommunizieren können und meinen, dass sie Teil des Quorums sind. Es ist daher sicherzustellen, dass
  *  die Quorum-Regel einen solchen Zustand vermeidet, also eine eindeutige Regel existiert, wer Teil des Quorums ist. 
  *  die Clusterimplementierung muss dafür sorge tragen, dass das Fencing erfolgreich durchgeführt wurde, damit das Quorum die Ressourcen des ausgefallen Knotens übernimmt.

Beteiligte Prozesse im Fencing:
corosync ist jene Applikation, die das Clustermembership behandelt. Unterbrechnung in der Kommunikation zwischen Knoten werden von corosync identifiziert. 
Pacemaker ist fungiert als Fencing/Stonith Manager. Die Pacemaker - Daemons fungieren als Ressource-Manager, haben Daemons um eine Ressource wieder in einen stabilen Zustand zu versetzen, die Cluterkonfigurtion managenen usw. Im Pacemaker-Paket enthalten ist auch ein stonith-ng (stonith next generation) Agent, welcher von anderen Pacemaker-Agenten angesprochen werden kann, um administrative Fencing-Befehle abzusetzen.

Sogenannte *fence-agents* sind Programme oder Scripte, die mit dem Fencing-Device interagieren können, um ggfs. ein Fencing auszulösen.

sbd steht für Storage-based-death" und unterstützt verschiedene Methd and provides several different methods for achieving "self fencing" - a fencing strategy that enlists a node to control its own state and its own access to shared resources. Despite its name, not all of the sbd methods depend on storage, but one method uses shared storage devices to communicate fencing instructions throughout a cluster.







