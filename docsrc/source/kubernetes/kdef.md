# Definitionen

## Was ist Kubernetes
Kubernetes ist eine portable, erweiterbare Open-Source-Plattform zur Automatisierung von Bereitstellung, Skalierung und Verwaltung von containerisierten Workloads. Kubernetes abstrahiert die komplexe Containerverwaltung und bietet uns eine deklarative Konfiguration zur Orchestrierung von Containern in verschiedenen Compute-Umgebungen. 

## Clusterknoten
Cluster basieren auf Knoten. Es gibt zwei Arten von Knoten in einem Kubernetes-Cluster, die spezifische Funktionen bereitstellen.

**Steuerungsebenenknoten**: Diese Knoten hosten die Aspekte der Steuerungsebene des Clusters und sind für die Dienste reserviert, die den Cluster steuern. Sie sind dafür verantwortlich, die API bereitzustellen, die Sie und alle anderen Knoten zur Kommunikation verwenden. Auf diesen Knoten werden keine Workloads bereitgestellt oder geplant.

**Knoten**: Diese Knoten sind für die Ausführung benutzerdefinierter Workloads und Anwendungen verantwortlich, z. B. für die Komponenten Ihres cloudbasierten Videorenderingdiensts.

## Knotenpool

Ein Knotenpool beschreibt eine Gruppe von Knoten mit der gleichen Konfiguration in einem AKS-Cluster. Diese Knoten enthalten die zugrunde liegenden VMs, auf denen Ihre Anwendungen ausgeführt werden. Sie können zwei Typen von Knotenpools in einem von AKS verwalteten Kubernetes-Cluster erstellen:

* Systemknotenpools

* Benutzerknotenpools

### Systemknotenpools 

Systemknotenpools hosten kritische Systempods, die die Steuerungsebene des Clusters bilden. Ein Systemknotenpool ermöglicht nur die Verwendung von Linux als Knotenbetriebssystem und führt nur Linux-basierte Workloads aus. Knoten in einem Systemknotenpool sind für Systemworkloads reserviert und werden normalerweise nicht zum Ausführen von benutzerdefinierten Workloads verwendet. Jeder AKS-Cluster muss mindestens einen Systemknotenpool mit mindestens einem Knoten enthalten, und Sie müssen die zugrunde liegenden VM-Größen für Knoten definieren.

Beispielsweise ist es in einem Systemknotenpool von entscheidender Bedeutung, die Höchstanzahl von Pods, die auf einem einzelnen Knoten ausgeführt werden sollen, auf 30 festzulegen. Mit diesem Wert wird sichergestellt, dass ausreichend Speicherplatz verfügbar ist, um die Systempods auszuführen, die für die Clusterintegrität essenziell sind. Wenn die Anzahl der Pods diesen Mindestwert überschreitet, sind im Pool neue Knoten erforderlich, um zusätzliche Workloads zu planen. Aus diesem Grund benötigt der Systemknotenpool mindestens einen Knoten im Pool. Für Produktionsumgebungen beträgt die empfohlene Knotenanzahl für einen Systemknotenpool mindestens drei Knoten.

### Benutzerknotenpools

Benutzerknotenpools unterstützen Ihre Workloads und ermöglichen es Ihnen, Windows oder Linux als Knotenbetriebssystem anzugeben. Sie können auch die zugrunde liegenden VM-Größen für Knoten definieren und bestimmte Workloads ausführen.
Es kann mehrere Benutzerknotenpools geben, z.B. nach Verwendung oder Technologie separiert. 
Sie können in einem Knotenpool bis zu 100 Knoten konfigurieren. Die Anzahl der Knoten, die Sie konfigurieren möchten, hängt jedoch von der Anzahl der Pods ab, die pro Knoten ausgeführt werden.
Benutzerknotenpools sind so konzipiert, dass sie benutzerdefinierte Workloads ausführen und nicht die 30-Pod-Anforderung aufweisen. Mit Benutzerknotenpools können Sie die Knotenanzahl für einen Pool auf null festlegen.
Beachten Sie, dass Namen von Knotenpools mit einem Kleinbuchstaben beginnen müssen und nur alphanumerische Zeichen enthalten dürfen. Knotenpoolnamen sind auf zwölf Zeichen für Linux-Knotenpools und sechs Zeichen für Windows-Knotenpools beschränkt.

## Was ist ein Container?
Ein Container ist eine kleine Einheit von Software, die Code, Abhängigkeiten und Konfiguration für eine bestimmte Anwendung bündelt. Mit Containern können Sie monolithische Anwendungen in einzelne Services aufteilen, aus denen sich die Lösung zusammensetzt.

## Pods
Ein Kubernetes-Pod gruppiert Container und Anwendungen in eine logische Struktur. Diese Pods verfügen über keine Intelligenz und bestehen aus mindestens einem Anwendungscontainer. Jeder verfügt über eine IP-Adresse, Netzwerkregeln und verfügbar gemachte Ports.

## Deployment
Eine Kubernetes-Bereitstellung ist eine Weiterentwicklung von Pods. Eine Bereitstellung umschließt die Pods in einem intelligenten Objekt, wodurch das horizontale Skalieren ermöglicht wird. Sie können Ihre Anwendung problemlos duplizieren und skalieren, um eine höhere Auslastung zu unterstützen, ohne dass Sie komplexe Netzwerkregeln konfigurieren müssen.

## Kubernetes-Manifestdatei
Mit einer Kubernetes-Manifestdatei können Sie Ihre Workloads im YAML-Format deklarativ beschreiben und die Kubernetes-Objektverwaltung vereinfachen.

Beispiel einer Manifestdatei für eine Ressource

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: contoso-website # This will be the name of the deployment
```

Bei Bereitstellungen wird eine label verwendet, um nach Pods zu suchen und diese zu gruppieren. Sie definieren die Bezeichnung als Teil der Manifestdatei Ihrer Bereitstellung.

```yaml
# deployment.yaml
# ...
spec:
  selector:
    matchLabels:
      app: contoso-website
# ...
```

## Kubernetes Zugangscontroller

Ein Zugangscontroller ist ein Kubernetes-Plug-In, das authentifizierte und autorisierte Anforderungen an die Kubernetes-API abfängt, bevor das angeforderte Kubernetes-Objekt persistent gespeichert wird. Angenommen, Sie stellen beispielsweise eine neue Workload bereit und die Bereitstellung enthält eine Pod-Anforderung mit spezifischen Arbeitsspeicheranforderungen. Der Zugangscontroller fängt die Bereitstellungsanforderung ab und muss die Bereitstellung autorisieren, bevor sie im Cluster gespeichert wird

### Zugangscontroller Web-Hook

Ein Zugangscontroller-Webhook ist eine HTTP-Rückruffunktion, die Zugangsanforderungen empfängt und diese Anforderungen dann verarbeitet. Die Steuerungscontroller müssen zur Laufzeit konfiguriert werden. Diese Controller sind entweder für Ihr kompiliertes Zugangs-Plug-In oder für Ihre bereitgestellte Erweiterung verfügbar, die als Webhook ausgeführt wird.

Es sind zwei Arten von Zugangscontroller-Webhooks verfügbar: ein überprüfender Webhook und ein verändernder Webhook. Ein verändernder Webhook wird zuerst aufgerufen und kann die Standardwerte der an den API-Server gesendeten Objekte ändern und anwenden. Ein Validierungswebhook überprüft Objektwerte und kann Anforderungen ablehnen.

## Open Policy Agent

Beim Open Policy Agent (OPA) handelt es sich um eine universelle Open-Source-Richtlinien-Engine, die eine allgemeine deklarative Sprache zum Erstellen von Richtlinien bietet. Diese Richtlinien ermöglichen es Ihnen, Regeln zu definieren, die das Verhalten Ihres Systems überwachen.

## Open Policy Agent Gatekeeper 

OPA Gatekeeper ist ein validierender Open-Source-Kubernetes-Zugangscontroller-Webhook, der CRD-basierte (Custom Resource Definition) Richtlinien erzwingt, die der OPA-Syntax folgen.

Das Ziel von OPA Gatekeeper besteht darin, Ihnen das Anpassen von Zugangsrichtlinien mithilfe der Konfiguration anstelle von hartcodierten Richtlinienregeln für Dienste zu ermöglichen. Außerdem erhalten Sie einen vollständigen Überblick über Ihren Cluster, um Ressourcen mit Richtlinienverletzungen zu identifizieren.

Verwenden Sie OPA Gatekeeper, um organisationsweite Richtlinien mit Regeln zu definieren:

* Dass die maximalen Ressourcenlimits, z. B. die CPU- und Arbeitsspeicherlimits, für alle konfigurierten Pods erzwungen werden

* Dass die Bereitstellung von Images nur aus genehmigten Repositorys zulässig ist

* Die Namenskonvention für Bezeichnungen für alle Namespaces in einem Cluster muss einen Kontaktpunkt für jeden Namespace angeben.

* Legen Sie fest, dass Clusterdienste global eindeutige Selektoren haben müssen.





Bridge to Kubernetes -> testen, plugin in vsc. 

