# AKS

## Verwalten von Knotenpools
https://learn.microsoft.com/de-de/training/modules/aks-optimize-compute-costs/2-node-pools

### Manuelle Skalierung von Knotenpools

Wenn Sie Workloads ausführen, die für eine bestimmte Dauer in bestimmten bekannten Intervallen ausgeführt werden, ist die manuelle Skalierung der Knotenpoolgröße eine gute Möglichkeit, die Knotenkosten zu steuern.

```
az aks nodepool add 
  --resource-group resourceGroup 
  --cluster-name aksCluster 
  --name gpunodepool 
  --node-count 1 
  --node-vm-size Standard_NC6 
  --no-wait
```

Nach dem Load kann man wieder auf 0 skalieren 

```
az aks nodepool scale 
  --resource-group resourceGroup 
  --cluster-name aksCluster 
  --name gpunodepool 
  --node-count 0
```

### Automatische Skalierung von Knotenpools

Hier gibt es zwei Möglichkeiten 

1. automatische horizontale Podskalierung

2. automatische Clusterskalierung

#### automatische horizontale Podskalierung

Die automatische horizontale Podskalierung von Kubernetes dient zum Überwachen des Ressourcenbedarfs in einem Cluster und zum automatischen Skalieren der Anzahl von Workloadreplikaten.

Der Metrikserver in Kubernetes erfasst Speicher- und Prozessormetriken von Controllern, Knoten und Containern, die im AKS-Cluster ausgeführt werden. Eine der Möglichkeiten, auf diese Informationen zuzugreifen, ist die Verwendung der Metrik-API. Die automatische horizontale Skalierung überprüft die Metrik-API alle 30 Sekunden, um zu entscheiden, ob Ihre Anwendung zusätzliche Instanzen benötigt, um dem erforderlichen Bedarf zu genügen.

Allerdings skaliert die automatische horizontale Podskalierung nur Pods auf verfügbaren Knoten in den konfigurierten Knotenpools des Clusters.

#### automatische Clusterskalierung

Eine Ressourceneinschränkung wird ausgelöst, wenn die automatische horizontale Podskalierung keinen weiteren Pod auf den vorhandenen Knoten in einem Knotenpool planen kann. Sie müssen in Zeiten von Einschränkungen die automatische Clusterskalierungs-Funktion zum Skalieren der Anzahl der Knoten in den Knotenpools eines Clusters verwenden. Die automatische Clusterskalierung überprüft die definierten Metriken und skaliert die Anzahl von Knoten basierend auf den erforderlichen Computeressourcen hoch oder herunter.

Die automatische Clusterskalierung wird zusammen mit der automatischen horizontalen Podskalierung verwendet.

Die automatische Clusterskalierung überwacht sowohl das zentrale Hochskalieren als auch das Herunterskalieren und ermöglicht es dem Kubernetes-Cluster, die Knotenanzahl in einem Knotenpool zu ändern, wenn sich der Ressourcenbedarf ändert.

Sie konfigurieren jeden Knotenpool mit anderen Skalierungsregeln. Beispielsweise könnten Sie nur einen Knotenpool so konfigurieren, dass die automatische Skalierung zugelassen wird. Sie könnten aber auch einen Knotenpool so konfigurieren, dass er nur auf eine bestimmte Anzahl von Knoten skaliert wird.

**!! Wichtig !!**

Sie verlieren die Fähigkeit, die Knotenanzahl auf null zu skalieren, wenn Sie die automatische Clusterskalierung für einen Knotenpool aktivieren. Stattdessen können Sie die Mindestanzahl auf null festlegen, um Clusterressourcen zu sparen.

### Spot VM

Bei einem virtuellen Spot-Computer handelt es sich um eine VM, die Ihnen Zugriff auf ungenutzte Azure-Computekapazität mit hohen Rabatten ermöglicht. Spot-VMs ersetzen die vorhandenen VMs mit niedriger Priorität in Azure. Sie können Spot-VMs zum Ausführen von Workloads verwenden, die Folgendes umfassen:

* High-Performance-Computing-Szenarios, Batchverarbeitung oder visuelle Renderinganwendungen

* umfangreiche zustandslose Anwendungen

* Entwickler-/Testumgebungen, einschließlich CI- (Continuous Integration) und CD-Workloads (Continuous Delivery)

Die Standardentfernungsrichtlinie für die Spot-VM-Entfernung ist Zuordnung aufheben. Azure räumt Spot-VMs mit einer Vorankündigung von 30 Sekunden, wenn die Kapazität in einer Region begrenzt wird. Eine VM, die mit der Richtlinie Zuordnung aufheben festgelegt ist, wechselt bei der Entfernung in den Zustand „stopped-deallocated“. Sie können eine entfernte VM erneut bereitstellen, wenn die Spot-Kapazität wieder verfügbar wird. Eine VM, deren Zuordnung aufgehoben wird, zählt weiterhin zu Ihrem Spot-vCPU-Kontingent, und es fallen weiterhin Gebühren für die zugrunde liegenden zugeordneten Datenträger an.

Eine **Spot-VM-Skalierungsgruppe** ist eine VM-Skalierungsgruppe, die Azure-Spot-VMs unterstützt. Diese VMs weisen dasselbe Verhalten wie normale Spot-VMs auf, jedoch mit einem Unterschied: Wenn Sie VM-Skalierungsgruppenunterstützung für Spot-VMs in Azure verwenden, müssen Sie sich zwischen zwei Entfernungsrichtlinien entscheiden:

* Zuordnung aufheben

* Löschen: Mit der Richtlinie „Löschen“ können Sie Kosten für Datenträger und das Erreichen von Kontingentgrenzen vermeiden. Mit der Entfernungsrichtlinie „Löschen“ werden entfernte VMs zusammen mit den zugrunde liegenden Datenträgern gelöscht. 

Als Best Practice gilt, das Feature für automatische Skalierung nur dann zu verwenden, wenn Sie die Entfernungsrichtlinie für die Skalierungsgruppe auf Löschen festlegen.

### Spot Knotenpool

Ein Spot-Knotenpool ist ein Benutzerknotenpool, der eine Spot-VM-Skalierungsgruppe verwendet. AKS unterstützt Spot-VMs in folgenden Fällen:

* Sie müssen Benutzerknotenpools erstellen.
* Sie möchten von den von der VM-Skalierungsgruppenunterstützung für Spot-VMs in Azure gebotenen Vorteilen profitieren.

Einschränkungen: 
* Die zugrunde liegende Spot-Skalierungsgruppe wird nur in einer einzelnen Fehlerdomäne bereitgestellt und bietet keine Garantien für Hochverfügbarkeit.
* Für den AKS-Cluster muss Unterstützung mehrerer Knotenpools aktiviert werden.
* Sie können Spot-Knotenpools nur als Benutzerknotenpools verwenden.
* Spot-Knotenpools können nicht aktualisiert werden.
* Die Erstellung von Spot-VMs wird nicht garantiert. Die Erstellung von Spot-Knoten hängt von der Kapazität- und Kontingentverfügbarkeit in der bereitgestellten Azure-Region des Clusters ab.

Beispielkonfiguration: 
``` 
az aks nodepool add 
  --resource-group resourceGroup 
  --cluster-name aksCluster 
  --name spotpool01 
  --enable-cluster-autoscaler 
  --max-count 3 
  --min-count 1 
  --priority Spot 
  --eviction-policy Delete 
  --spot-max-price -1 
  --no-wait
```

#### Bereitstellung von Pods auf Spot-Knotenpools

Wenn Sie Workloads in Kubernetes bereitstellen, können Sie Informationen für den Scheduler bereitstellen, um anzugeben, auf welchen Knoten die Workloads ausgeführt werden können und auf welchen nicht. Sie steuern die Workloadplanung durch Konfigurieren von Markierungen, Toleranz oder Knotenaffinität.

Ein Taint ist ein Merkmal, um Pods abzulehnen auf dem Knoten zu laufen (opposite of affinity)[https://kubernetes.io/docs/concepts/scheduling-eviction/taint-and-toleration/]. Spot-Knoten werden mit einer Bezeichnung konfiguriert, die auf kubernetes.azure.com/scalesetpriority:spot festgelegt ist.

Tolerations are applied to pods. Tolerations allow the scheduler to schedule pods with matching taints. 

Sie verwenden die Knotenaffinität, um zu beschreiben, welche Pods auf einem Knoten geplant werden. Affinität wird mithilfe von Bezeichnungen angegeben, die auf dem Knoten definiert werden. In AKS werden Systempods z. B. mit **Anti**affinität für Spot-Knoten konfiguriert, um zu verhindern, dass sie auf diesen Knoten geplant werden.

Beispiel einer Manifestdatei für ein Pod

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx
  labels:
    env: test
spec:
  containers:
  - name: nginx
    image: nginx
    imagePullPolicy: IfNotPresent
  tolerations:  <-- auf diesem Knoten kann ein Pod mit kubernetes.azure.com/scalesetpriority = spot laufen
  - key: "kubernetes.azure.com/scalesetpriority"
    operator: "Equal"
    value: "spot"
    effect: "NoSchedule"
  affinity:
    nodeAffinity:   <-- auf welchen Node möchte ich laufen kubernetes.azure.com/scalesetpriority = spot
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
        - matchExpressions:
          - key: "kubernetes.azure.com/scalesetpriority"
            operator: In
            values:
            - "spot"
```

## Azure Policy für AKS

Azure Policy erweitert OPA Gatekeeper-Version 3 und wird über integrierte Richtlinien mit AKS integriert. Diese Richtlinien wenden maßgerechte Erzwingungen und Schutzvorrichtungen auf zentralisierte und konsistente Weise auf Ihren Cluster an.

### Aktivieren des Azure Policy Add Ons für AKS

https://learn.microsoft.com/de-de/training/modules/aks-optimize-compute-costs/6-resource-quota-azure-policy


