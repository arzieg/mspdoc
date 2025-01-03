# Definitionen

## Was ist Kubernetes
Kubernetes ist eine portable, erweiterbare Open-Source-Plattform zur Automatisierung von Bereitstellung, Skalierung und Verwaltung von containerisierten Workloads. Kubernetes abstrahiert die komplexe Containerverwaltung und bietet uns eine deklarative Konfiguration zur Orchestrierung von Containern in verschiedenen Compute-Umgebungen. 

## Clusterknoten
Cluster basieren auf Knoten. Es gibt zwei Arten von Knoten in einem Kubernetes-Cluster, die spezifische Funktionen bereitstellen.

**Steuerungsebenenknoten**: Diese Knoten hosten die Aspekte der Steuerungsebene des Clusters und sind für die Dienste reserviert, die den Cluster steuern. Sie sind dafür verantwortlich, die API bereitzustellen, die Sie und alle anderen Knoten zur Kommunikation verwenden. Auf diesen Knoten werden keine Workloads bereitgestellt oder geplant.

**Knoten**: Diese Knoten sind für die Ausführung benutzerdefinierter Workloads und Anwendungen verantwortlich, z. B. für die Komponenten Ihres cloudbasierten Videorenderingdiensts.

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




Bridge to Kubernetes -> testen, plugin in vsc. 

