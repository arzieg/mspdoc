# Kubernetes

Quick Reference: https://kubernetes.io/docs/reference/kubectl/quick-reference/

## Port Forwarding
```
kubectl port-forward -n vault service/vault-active 8200:8200
kubectl port-forward -n vault service/vault 8200:8200
```

Get Ingress Services:  `kubectl get ingress -A`

## Secrets

Secrets für WebUI etc. sind häufig nicht bekannt aber in Kubernetes gespeichert. 

Secrets anzeigen: `kubectl get secrets -A`
Einzelnes Secret anzeigen: `kubectl get secret -n argocd argocd-initial-admin-secret -o yaml`

Das Passwort ist base64 verschlüsselt, dass muss man dann wieder umwandeln
`echo <Passwort> | base64 -d`






## Custom Resource

Custom resources are extensions of the Kubernetes API. A custom resource is an extension of the Kubernetes API that is not necessarily available in a default Kubernetes installation. 

Liste der CRD: `kubectl get crd`
Beschreibung der CRD:  `kubectl get crd applications.argoproj.io -o yaml`
Aktualisierung einer CRD:  `kubectl apply --server-side -k "github.com/ansible/awx-operator/config/crd?ref=2.19.0"

Im diesem Beispiel ist im gitrepo awx-operator im Verzeichnis config/crd ein git-tag 2.19.0 mit einer akutellen CRD Beschreibung für das Produkt (API Beschreibung). In der kustomization.yaml stehen dann die einzelnen API-Keys, deren Typ und zulässige Werte. Die können sich natürlich mit der Zeit ändern und müssen ggfs. aktualisiert werden.




