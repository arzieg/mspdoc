# Kubernetes

Quick Reference: https://kubernetes.io/docs/reference/kubectl/quick-reference/

alias k=kubectl

## Viewing and finding resources

-A = --all-namespaces 
-n = --namespace

### Get commands with basic output
kubectl get services                          # List all services in the namespace\
kubectl get pods --all-namespaces             # List all pods in all namespaces\
kubectl get pods -o wide                      # List all pods in the current namespace, with more details\
kubectl get deployment -n 'namespace'         # List a particular deployment\
kubectl get pods                              # List all pods in the namespace\
kubectl get pod my-pod -o yaml                # Get a pod's YAML\
kubectl get nodes -o wide                     # Get Node Information\
kubectl get rs -n 'namespace'                 # Get Replicaset of namespace\
kubectl get ing -n 'namespace'                # Get ingress controllers

    

### Describe commands with verbose output
kubectl describe nodes my-node
kubectl describe pods my-pod -n namespace     # describe pod

### List Services Sorted by Name
kubectl get services --sort-by=.metadata.name

### List pods Sorted by Restart Count
kubectl get pods --sort-by='.status.containerStatuses[0].restartCount'

### List PersistentVolumes sorted by capacity
kubectl get pv --sort-by=.spec.capacity.storage

### Get the version label of all pods with label app=cassandra
kubectl get pods --selector=app=cassandra -o jsonpath='{.items[*].metadata.labels.version}'

### Retrieve the value of a key with dots, e.g. 'ca.crt'
kubectl get configmap myconfig -o jsonpath='{.data.ca\.crt}'

### Retrieve a base64 encoded value with dashes instead of underscores.
kubectl get secret my-secret --template='{{index .data "key-name-with-dashes"}}'

### Get all worker nodes (use a selector to exclude results that have a label named 'node-role.kubernetes.io/control-plane')
kubectl get node --selector='!node-role.kubernetes.io/control-plane'

### Get all running pods in the namespace
kubectl get pods --field-selector=status.phase=Running

### Get ExternalIPs of all nodes
kubectl get nodes -o jsonpath='{.items[*].status.addresses[?(@.type=="ExternalIP")].address}'

### List Names of Pods that belong to Particular RC
### "jq" command useful for transformations that are too complex for jsonpath, it can be found at https://jqlang.github.io/jq/
sel=${$(kubectl get rc my-rc --output=json | jq -j '.spec.selector | to_entries | .[] | "\(.key)=\(.value),"')%?}
echo $(kubectl get pods --selector=$sel --output=jsonpath={.items..metadata.name})

### Show labels for all pods (or any other Kubernetes object that supports labelling)
kubectl get pods --show-labels

### Check which nodes are ready
JSONPATH='{range .items[*]}{@.metadata.name}:{range @.status.conditions[*]}{@.type}={@.status};{end}{end}' \
 && kubectl get nodes -o jsonpath="$JSONPATH" | grep "Ready=True"

### Check which nodes are ready with custom-columns
kubectl get node -o custom-columns='NODE_NAME:.metadata.name,STATUS:.status.conditions[?(@.type=="Ready")].status'

### Output decoded secrets without external tools
kubectl get secret my-secret -o go-template='{{range $k,$v := .data}}{{"### "}}{{$k}}{{"\n"}}{{$v|base64decode}}{{"\n\n"}}{{end}}'

### List all Secrets currently in use by a pod (das geht nicht)
kubectl get pods -o json | jq '.items[].spec.containers[].env[]?.valueFrom.secretKeyRef.name' | grep -v null | sort | uniq

### List all containerIDs of initContainer of all pods
### Helpful when cleaning up stopped containers, while avoiding removal of initContainers.
kubectl get pods --all-namespaces -o jsonpath='{range .items[*].status.initContainerStatuses[*]}{.containerID}{"\n"}{end}' | cut -d/ -f3

### List Events sorted by timestamp
kubectl get events --sort-by=.metadata.creationTimestamp -A

### List all warning events
kubectl events --types=Warning -n namespace

### Compares the current state of the cluster against the state that the cluster would be in if the manifest was applied.
kubectl diff -f ./my-manifest.yaml

### Produce a period-delimited tree of all keys returned for nodes
### Helpful when locating a key within a complex nested JSON structure
kubectl get nodes -o json | jq -c 'paths|join(".")'

### Produce a period-delimited tree of all keys returned for pods, etc
kubectl get pods -o json | jq -c 'paths|join(".")'

### Produce ENV for all pods, assuming you have a default container for the pods, default namespace and the `env` command is supported.
### Helpful when running any supported command across all pods, not just `env`
for pod in $(kubectl get po --output=jsonpath={.items..metadata.name}); do echo $pod && kubectl exec -it $pod -- env; done

### Get a deployment's status subresource
kubectl get deployment nginx-deployment --subresource=status


## Pod Management

k delete pod -n argocd 'pod-name' 
kubectl logs my-pod                                 # dump pod logs (stdout)\
kubectl logs -l name=myLabel                        # dump pod logs, with label name=myLabel (stdout)\
kubectl logs my-pod --previous                      # dump pod logs (stdout) for a previous instantiation of a container\
kubectl logs my-pod -c my-container                 # dump pod container logs (stdout, multi-container case)\
kubectl logs -l name=myLabel -c my-container        # dump pod container logs, with label name=myLabel (stdout)\
kubectl logs my-pod -c my-container --previous      # dump pod container logs (stdout, multi-container case) for a previous instantiation of a container\
kubectl logs -f my-pod                              # stream pod logs (stdout)\
kubectl logs -f my-pod -c my-container              # stream pod container logs (stdout, multi-container case)\
kubectl logs -f -l name=myLabel --all-containers    # stream all pods logs with label name=myLabel (stdout)\
kubectl run -i --tty busybox --image=busybox:1.28 -- sh  # Run pod as interactive shell\
kubectl run nginx --image=nginx -n mynamespace      # Start a single instance of nginx pod in the namespace of mynamespace\
kubectl run nginx --image=nginx --dry-run=client -o yaml > pod.yaml
                                                    # Generate spec for running pod nginx and write it into a file called pod.yaml\
kubectl attach my-pod -i                            # Attach to Running Container\
kubectl port-forward my-pod 5000:6000               # Listen on port 5000 on the local machine and forward to port 6000 on my-pod\
kubectl exec my-pod -- ls /                         # Run command in existing pod (1 container case)\
kubectl exec --stdin --tty my-pod -- /bin/sh        # Interactive shell access to a running pod (1 container case)\
kubectl exec my-pod -c my-container -- ls /         # Run command in existing pod (multi-container case)\
kubectl debug my-pod -it --image=busybox:1.28       # Create an interactive debugging session within existing pod and immediately attach to it\
kubectl debug node/my-node -it --image=busybox:1.28 # Create an interactive debugging session on a node and immediately attach to it\
kubectl top pod -n 'namespace'                      # Show metrics for all pods in the default namespace\
kubectl top pod POD_NAME --containers               # Show metrics for a given pod and its containers\
kubectl top pod POD_NAME --sort-by=cpu              # Show metrics for a given pod and sort it by 'cpu' or 'memory'\


## Port Forwarding

vault:
```
kubectl port-forward -n vault service/vault-active 8200:8200
kubectl port-forward -n vault service/vault 8200:8200
ssh -L 8200:localhost:8200 -J <user>@<jumphost> <adminuser>@<adminjumphost>
```

argocd:
```
kubectl port-forward -n argocd service/argocd-server 8080:80
ssh -L 8080:localhost:8080 -J <user>@<jumphost> <adminuser>@<adminjumphost>
```

Get Ingress Services:  `kubectl get ingress -A`

## Secrets

### Show secrets
Secrets anzeigen: `kubectl get secrets -A`
Einzelnes Secret anzeigen: `kubectl get secret -n argocd argocd-initial-admin-secret -o yaml`

Das Passwort ist base64 verschlüsselt, dass muss man dann wieder umwandeln
`echo <Passwort> | base64 -d`

### delete secrets

k delete secret 'podname' -n argocd 








## Custom Resource

Custom resources are extensions of the Kubernetes API. A custom resource is an extension of the Kubernetes API that is not necessarily available in a default Kubernetes installation. 

Liste der CRD: `kubectl get crd`
Beschreibung der CRD:  `kubectl get crd applications.argoproj.io -o yaml`
Aktualisierung einer CRD:  `kubectl apply --server-side -k "github.com/ansible/awx-operator/config/crd?ref=2.19.0"

Im diesem Beispiel ist im gitrepo awx-operator im Verzeichnis config/crd ein git-tag 2.19.0 mit einer akutellen CRD Beschreibung für das Produkt (API Beschreibung). In der kustomization.yaml stehen dann die einzelnen API-Keys, deren Typ und zulässige Werte. Die können sich natürlich mit der Zeit ändern und müssen ggfs. aktualisiert werden.


## Replica Set

kubectl delete rs -n 'namespace' 'podname'

## Deployment

k get deploy -n 'namespace'
k delete deploy -n 'namespace' 'deploymentname'
k apply -f my-deployment.yaml



## Interacting with Nodes and cluster

kubectl cordon my-node                                                # Mark my-node as unschedulable\
kubectl drain my-node                                                 # Drain my-node in preparation for maintenance\
kubectl uncordon my-node                                              # Mark my-node as schedulable\
kubectl top node                                                      # Show metrics for all nodes\
kubectl top node my-node                                              # Show metrics for a given node\
kubectl cluster-info                                                  # Display addresses of the master and services\
kubectl cluster-info dump                                             # Dump current cluster state to stdout\
kubectl cluster-info dump --output-directory=/path/to/cluster-state   # Dump current cluster state to /path/to/cluster-state\

### View existing taints on which exist on current nodes.
kubectl get nodes -o='custom-columns=NodeName:.metadata.name,TaintKey:.spec.taints[*].key,TaintValue:.spec.taints[*].value,TaintEffect:.spec.taints[*].effect'

### If a taint with that key and effect already exists, its value is replaced as specified.
kubectl taint nodes foo dedicated=special-user:NoSchedule
