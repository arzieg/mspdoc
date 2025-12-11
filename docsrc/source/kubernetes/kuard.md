# Kubernetes Up and Running


# Pods

A Pod is a collection of application containers and volumes running in the same execution environment. Pods, not containers, are the smallest deployable artifact in a
Kubernetes cluster. This means all of the containers in a Pod always land on the same machine. Each container within a Pod runs in its own cgroup, but they share a number of
Linux namespaces.

Applications running in the same Pod share the same IP address and port space (network namespace), have the same hostname (UTS namespace), and can communicate using native interprocess communication channels over System V IPC or POSIX message queues (IPC namespace). However, applications in different Pods are isolated from each other; they have different IP addresses, hostnames, and more.

In general, the right question to ask yourself when designing Pods is “Will these containers work correctly if they land on different machines?” If the answer is no,
a Pod is the correct grouping for the containers. If the answer is yes, using multiple Pods is probably the correct solution.

## The Pod Manifest

Pods are described in a Pod manifest, which is just a text-file representation of the Kubernetes API object.

## Creating a pod

```
kubectl run kuard --image=gcr.io/kuar-demo/kuard-amd64:blue
kubectl get pods
kuebectl delete pods/kuard
```

## Healthcheck

When you run your application as a container in Kubernetes, it is automatically kept alive for you using a process health check. This health check simply ensures that the main process of your application is always running. If it isn’t, Kubernetes restarts it. Liveness probes are defined per container, which means each container inside a Pod is health checked separately.

```
apiVersion: v1
kind: Pod
metadata:
 name: kuard
spec:
 containers:
 - image: gcr.io/kuar-demo/kuard-amd64:blue
   name: kuard
   livenessProbe:
     httpGet:
        path: /healthy
        port: 8080
     initialDelaySeconds: 5
     timeoutSeconds: 1
     periodSeconds: 10
     failureThreshold: 3
   ports:
     - containerPort: 8080
       name: http
       protocol: TCP
 ```

Unterschieden wird zwischen
* **Liveness** determines if an application is running properly
* **Readiness** Readiness describes when a container is ready to serve user requests.
* **Starup probe** Startup probes have recently been introduced to Kubernetes as an alternative way of managing slow-starting containers. When a Pod is started, the startup robe is run before any other probing of the Pod is started
* **Advanced probe** 

HealthChecks können sein
* http/https
* tcpSocket
* exec (eines beliebigen Scriptes mit RC=0)


## Volumes

```
apiVersion: v1
kind: Pod
metadata:
  name: kuard
spec:
  volumes:
    - name: "kuard-data"
      hostPath:
        path: "/var/lib/kuard"
  containers:
    - image: docker.io/ksaito1125/kuard
      name: kuard
      resources:
        requests:
          cpu: "500m"
          memory: "128Mi"
        limits:
          cpu: "1000m"
          memory: "256Mi"
      livenessProbe:
        httpGet:
          path: /healthy
          port: 8080
        initialDelaySeconds: 5
        timeoutSeconds: 1
        periodSeconds: 10
        failureThreshold: 3
      ports:
        - containerPort: 8080
          name: http
          protocol: TCP
```

Typische Verwendung von Volumes in Kubernetes:
* communication/synchronization: zwei Container sharen sich das selbe volume
* cache: Das Volume wird als Cache verwendet
* persistent data: Daten auf die nach dem Neustart eines Containers wieder zugeriffen werden soll (Anbindung per nfs, iscsi, azure file, azure disk)
* mounting host filesystem: hier wird das Filesystem von Worker eingehängt, z.b. /dev (via. hostPath: /dev), Beispiel nfs:
    ```
    # Rest of pod definition above here
    volumes:
    - name: "kuard-data"
      nfs:
        server: my.nfs.server.local
        path: "/exports"
    ```

# Labels and Annotations

Labels and annotations are fundamental concepts in Kubernetes that let you work in sets of things that map to how you think about your application

* Labels are key/value pairs that can be attached to Kubernetes objects such as Pods and ReplicaSets. They can be arbitrary and are useful for attaching identifying information to Kubernetes objects. Labels provide the foundation for grouping objects.

* Annotations, on the other hand, provide a storage mechanism that resembles labels: key/value pairs designed to hold nonidentifying information that tools and libraries can leverage. Unlike labels, annotations are not meant for querying, filtering, or otherwise differentiating Pods from each other.

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: alpaca-test
  labels:                  <--- Label hier auf Ebene Deployment
    ver: "2"
    app: alpaca
    env: test
spec:
  replicas: 1
  selector:
    matchLabels:
      app: alpaca
  template:
    metadata:
      labels:
        app: alpaca         <--- hier kann man das auch definieren, insb. sinnvoll wenn man später nach PODs filtern will
    spec:
      containers:
      - name: kuard
        image: docker.io/ksaito1125/kuard
        ports:
        - containerPort: 8080
```

`kubectl get deployments --show-labels`

## Modifiying Labels

`kubectl label deployments alpaca-test "canary=true"`   - add label
`kubectl label deployments alpaca-test "canary-"`       - remove label


```
get deployments -L canary
NAME                READY   UP-TO-DATE   AVAILABLE   AGE     CANARY
alpaca-prod         2/2     2            2           10m
alpaca-test         1/1     1            1           10m     true
bandicoot-prod      2/2     2            2           3m7s
bandicoot-staging   1/1     1            1           3m47s
```

## Label Selectors aka Filter

```
get pods --show-labels --selector="ver=2"
NAME                                 READY   STATUS    RESTARTS   AGE    LABELS
alpaca-test-86c87767fb-h2rkz         1/1     Running   0          10m    app=alpaca,env=test,pod-template-hash=86c87767fb,ver=2
bandicoot-prod-597c79d778-bbbkw      1/1     Running   0          113s   app=bandicoot,env=prod,pod-template-hash=597c79d778,ver=2
bandicoot-prod-597c79d778-gdb5z      1/1     Running   0          113s   app=bandicoot,env=prod,pod-template-hash=597c79d778,ver=2
bandicoot-staging-6f68569b79-f79s8   1/1     Running   0          46s    app=bandicoot,env=stagging,pod-template-hash=6f68569b79,ver=2
```

logical AND (--selector="app=bandicoot,ver=2")

```
kubectl get pods --selector="app=bandicoot,ver=2"
NAME                                 READY   STATUS    RESTARTS   AGE
bandicoot-prod-597c79d778-bbbkw      1/1     Running   0          2m23s
bandicoot-prod-597c79d778-gdb5z      1/1     Running   0          2m23s
bandicoot-staging-6f68569b79-f79s8   1/1     Running   0          76s
```

logical OR (--selector="app in (alpaca,bandicoot))

``` 
kubectl get pods --selector="app in (alpaca,bandicoot)"
```

### Selector operators

| **Operator**               | **Description**                    |
| :------------------------- | ---------------------------------- |
| key=value                  | key is set to value                |
| key!=value                 | key is not set to value            |
| key in (value1, value2)    | key is one of value1 or value2     |
| key notin (value1, value2) | key is not one of value1 or value2 |
| key                        | key is set                         |
| !key                       | key is not set                     |


```
kubectl get pods --show-labels -l 'ver=2,!canary'
NAME                                 READY   STATUS    RESTARTS   AGE   LABELS
bandicoot-prod-597c79d778-bbbkw      1/1     Running   0          11m   app=bandicoot,env=prod,pod-template-hash=597c79d778,ver=2
bandicoot-prod-597c79d778-gdb5z      1/1     Running   0          11m   app=bandicoot,env=prod,pod-template-hash=597c79d778,ver=2
bandicoot-staging-6f68569b79-f79s8   1/1     Running   0          10m   app=bandicoot,env=stagging,pod-template-hash=6f68569b79,ver=2
```


In YAML

```
...
selector:
 matchLabels:
   app: alpaca
 matchExpressions:
   - {key: ver, operator: In, values: [1, 2]}
...
```
This is an item in a list (matchExpressions) that is a map with three entries. The last entry (values) has a value that is a list with two items. All of the terms are evaluated as a logical AND

The only way to represent the != operator is to convert it to a NotIn expression with a single value.

## Labels in the Kubernetes Architecture

In addition to enabling users to organize their infrastructure, labels play a critical role in linking various related Kubernetes objects. Kubernetes is a purposefully decoupled
system. However, in many cases, objects need to relate to one another, and these relationships are defined by labels and label selectors.

Labels are a powerful and ubiquitous glue that holds a Kubernetes application together. Though your application will likely start out with a simple set of labels and queries, you should expect it to grow in size and sophistication with time.


## Annotations

Annotations provide a place to store additional metadata for Kubernetes objects where the sole purpose of the metadata is assisting tools and libraries.

When in doubt, add information to an object as an annotation and promote it to a label if you find yourself wanting to use it in a selector.

Annotations are used to:
* Keep track of a “reason” for the latest update to an object.
* Communicate a specialized scheduling policy to a specialized scheduler.
* Extend data about the last tool to update the resource and how it was updated (used for detecting changes by other tools and doing a smart merge).
* Attach build, release, or image information that isn’t appropriate for labels (may include a Git hash, timestamp, pull request number, etc.).
* Enable the Deployment object (see Chapter 10) to keep track of ReplicaSets that it is managing for rollouts.
* Provide extra data to enhance the visual quality or usability of a UI. For example, objects could include a link to an icon (or a base64-encoded version of an icon).
* Prototype alpha functionality in Kubernetes (instead of creating a first-class API field, the parameters for that functionality are encoded in an annotation).

Bsp.:

```
...
metadata:
  annotations:
    example.com/icon-url: "https://example.com/icon.png"
...
```

# Service Discovery

While the dynamic nature of Kubernetes makes it easy to run a lot of things, it creates problems when it comes to finding those things. Most of the traditional network infrastructure wasn’t built for the level of dynamism that Kubernetes presents.

Servicediscovery tools help solve the problem of finding which processes are listening at which addresses for which services.

## Service Object

Real service discovery in Kubernetes starts with a Service object. A Service object is a way to create a named label selector

```
apiVersion: v1
kind: Service
metadata:
  name: alpaca-prod-service
spec:
  selector:
    app: alpaca
  ports:
  - protocol: TCP
    port: 80
    targetPort: 8080
  type: ClusterIP
```

type ist hier interessant: 
 type=Loadbalancer, es wird eine externe (in azure public) IP erstellt
 type=ClusterIP, es wird keine public IP erstellt (Service ist nicht von außen erreichbar)
 type=NodePort, This exposes the service on a static port (e.g., 30000–32767) on each node’s IP, but does not assign an external IP.

Möchte man hingegen, dass ein interner Loadbalancer verwendet wird, gibt man diesen in den Metadaten an. Dieser ist dann Cloud-Spezifisch, hier unter annotations.

```
apiVersion: v1
kind: Service
metadata:
  name: alpaca-prod-service
  annotations:
    service.beta.kubernetes.io/azure-load-balancer-internal: "true"

spec:
  selector:
    app: alpaca
...
```


Endpoints: 

```
kubectl get endpoints alpaca-prod-service --watch
NAME                  ENDPOINTS                                             AGE
alpaca-prod-service   10.244.0.12:8080,10.244.0.195:8080,10.244.0.72:8080   9m6s
```

## kube-proxy and ClusterIP

apiserver->kube-proxy->ClusterIP->Backend1/2/3...

The kube-proxy watches for new services in the cluster via the API server. It then programs a set of iptables rules in the kernel of that host to rewrite the destinations of packets so they are directed at one of the endpoints for that service. If the set of endpoints for a service changes (due to Pods coming and going or due to a failed readiness check), the set of iptables rules is rewritten.

The cluster IP itself is usually assigned by the API server as the service is created. However, when creating the service, the user can specify a specific cluster IP. Once
set, the cluster IP cannot be modified without deleting and re-creating the Service object.

!!!!!!

*The Kubernetes service address range is configured using the --service-cluster-ip-range flag on the kube-apiserver binary. The service address range should not overlap with theIP subnets and ranges assigned to each Docker bridge or Kubernetes node. In addition, any explicit cluster IP requested must come from that range and not already be in use.*

!!!!!!!

## Connecting to Resources Outside of a Cluster

When you are connecting Kubernetes to legacy resources outside of the cluster, you can use selector-less services to declare a Kubernetes service with a manually assigned IP address that is outside of the cluster. That way, Kubernetes service discovery via DNS works as expected, but the network traffic itself flows to an external resource.

To create a selector-less service, you remove the spec.selector field from your resource, while leaving the metadata and the ports sections unchanged.Because your service has no selector, no endpoints are automatically added to the service. This means that you must add them manually. Typically the endpoint that you will add will be a fixed IP address (e.g., the IP address of your database server) so you only need to add it once. But if the IP address that backs the service ever changes, you will need to update the corresponding endpoint resource. To create or update the endpoint resource, you use an endpoint that looks something like the following:

```
apiVersion: v1
kind: Endpoints
metadata:
  # This name must match the name of your service
  name: my-database-server
subsets:
  - addresses:
      # Replace this IP with the real IP of your server
     - ip: 1.2.3.4
    ports:
      # Replace this port with the port(s) you want to expose
      - port: 1433
...
 ```

## Connecting External Resources to Services Inside a Cluster

If your cloud provider supports it, the easiest thing to do is to create an “internal” load balancer, as described above, that lives in your virtual private network and can deliver traffic from a fixed IP address into the cluster. You can then use traditional DNS to make this IP address available to the external resource. If an internal load balancer isn’t available, you can use a NodePort service to expose the service on the IP addresses of the nodes in the cluster. You can then either program a physical load balancer to serve traffic to those nodes, or use DNS-based load-balancing to spread traffic between the nodes.

If neither of those solutions works for your use case, more complex options include running the full kube-proxy on an external resource and programming that machine to use the DNS server in the Kubernetes cluster. Such a setup is significantly more difficult to get right and should really only be used in on-premise environments. There are also a variety of open source projects (for example, HashiCorp’s Consul) that can be used to manage connectivity between in-cluster and out-of-cluster resources. Such options require significant knowledge of both networking and Kubernetes to get right and should really be considered a last resort.

# HTTP Load Balancing with Ingress

The Service object operates at Layer 4 (according to the OSI model). This means that it only forwards TCP and UDP connections and doesn’t look inside of those connections. Because of this, hosting many applications on a cluster uses many different exposed services. In the case where these services are type: NodePort, you’ll have to have clients connect to a unique port per service. In the case where these services are type: LoadBalancer, you’ll be allocating (often expensive or scarce) cloud resources for each service. 

But for HTTP (Layer 7)-based services, we can do better. When solving a similar problem in non-Kubernetes situations, users often turn to the idea of “virtual hosting.” This is a mechanism to host many HTTP sites on a single IP address. Typically, the user uses a load balancer or reverse proxy to accept incoming connections on HTTP (80) and HTTPS (443) ports. That program then parses the HTTP connection and, based on the Host header and the URL path that is requested, proxies the HTTP call to some other program. In this way, that load balancer or reverse proxy directs traffic for decoding and directing incoming connections to the right “upstream” server.

Kubernetes calls its HTTP-based load-balancing system **Ingress**. Ingress is a Kubernetes-native way to implement the “virtual hosting” pattern we just discussed. One of the more complex aspects of the pattern is that the user has to manage the load balancer configuration file. In a dynamic environment and as the set of virtual hosts expands, this can be very complex. The Kubernetes Ingress system works to simplify this by (a) standardizing that configuration, (b) moving it to a standard Kubernetes object, and (c) merging multiple Ingress objects into a single config for the load balancer.

The Ingress controller is a software system made up of two parts. 
* The first is the Ingress proxy, which is exposed outside the cluster using a service of type: LoadBalancer. This proxy sends requests to “upstream” servers. 
* The other component is the Ingress reconciler, or operator. The Ingress operator is responsible for reading and monitoring Ingress objects in the Kubernetes API and reconfiguring the Ingress proxy to route traffic as specified in the Ingress resource. 

## Ingress Spec

The Spec is split into a common resource specification and a controller implementation. There is no "standard" Ingress controller that is built into Kubernetes, so the user must install one of many optional implementations.

Users can create and modify Ingress objects just like every other object. But, by default, there is no code running to actually act on those objects.

### Beispiel Ingress Controller Contour

This is a controller built to configure the open source (and CNCF project) load balancer called Envoy. Envoy is built to be dynamically configured via an API. 

Deployment: 
```
kubectl apply -f https://projectcontour.io/quickstart/contour.yaml

bzw. 

kubectl apply -f .\contour.yaml.txt
namespace/projectcontour created
serviceaccount/contour created
serviceaccount/envoy created
configmap/contour created
customresourcedefinition.apiextensions.k8s.io/contourconfigurations.projectcontour.io configured
customresourcedefinition.apiextensions.k8s.io/contourdeployments.projectcontour.io configured
customresourcedefinition.apiextensions.k8s.io/extensionservices.projectcontour.io configured
customresourcedefinition.apiextensions.k8s.io/httpproxies.projectcontour.io configured
customresourcedefinition.apiextensions.k8s.io/tlscertificatedelegations.projectcontour.io configured
serviceaccount/contour-certgen created
rolebinding.rbac.authorization.k8s.io/contour created
role.rbac.authorization.k8s.io/contour-certgen created
job.batch/contour-certgen-v1-33-0 created
clusterrolebinding.rbac.authorization.k8s.io/contour unchanged
rolebinding.rbac.authorization.k8s.io/contour-rolebinding created
clusterrole.rbac.authorization.k8s.io/contour unchanged
role.rbac.authorization.k8s.io/contour created
service/contour created
service/envoy created
deployment.apps/contour created
daemonset.apps/envoy created
```

Wenn man keine Public IP Adresse definieren möchte (aka Loadbalancer), dann muss man bei der Containerdefinition von envoy ClusterIP oder NodePort angeben. 

``` 
apiVersion: v1
kind: Service
metadata:
  name: envoy
  namespace: projectcontour
  annotations:
    # This annotation puts the AWS ELB into "TCP" mode so that it does not
    # do HTTP negotiation for HTTPS connections at the ELB edge.
    # The downside of this is the remote IP address of all connections will
    # appear to be the internal address of the ELB. See docs/proxy-proto.md
    # for information about enabling the PROXY protocol on the ELB to recover
    # the original remote IP address.
    service.beta.kubernetes.io/aws-load-balancer-backend-protocol: tcp
spec:
  #externalTrafficPolicy: Local
  ports:
  - port: 80
    name: http
    protocol: TCP
    targetPort: 8080
  - port: 443
    name: https
    protocol: TCP
    targetPort: 8443
  selector:
    app: envoy
  type: ClusterIP <-- hier
```

IP Adresse ermitteln
```
kubectl get -n projectcontour service envoy -o wide
NAME    TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)          AGE   SELECTOR
envoy   ClusterIP   10.0.163.230   <none>        80/TCP,443/TCP   32s   app=envoy
```


### DNS konfigurieren

You need to configure DNS entries to the external address for your load balancer. You can map multiple hostnames to a single external endpoint
and the Ingress controller will direct incoming requests to the appropriate upstream service based on that hostname.

### Simple Ingress

```
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
 name: simple-ingress
spec:
 defaultBackend:
   service:
     name: alpaca
     port:
       number: 8080
```

```
kubectl apply -f simple-ingress.yaml
kubectl get ingress
kubectl describe ingress simple-ingress
```

### Ingress using Hostnames

The most common example of this is to have the Ingress system look at the HTTP host header (which is set to the DNS domain in the original URL) and direct traffic based on that header.

```
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: host-ingress
spec:
  defaultBackend:
    service:
      name: be-default
      port:
        number: 8080
  rules:   
    - host: alpaca.example.com
      http:
        paths:
          - pathType: Prefix
            path: /
            backend:
              service:
                name: alpaca
                port:
                  number: 8080
```

```
kubectl get ingress
kubectl describe ingress host-ingress


Name:             host-ingress
Labels:           <none>
Namespace:        default
Address:
Ingress Class:    <none>
Default backend:  be-default:8080 (10.244.0.192:8080,10.244.0.49:8080,10.244.0.50:8080)
Rules:
  Host                Path  Backends
  ----                ----  --------
  alpaca.example.com
                      /   alpaca:8080 (10.244.0.154:8080,10.244.0.233:8080,10.244.0.114:8080)
Annotations:          <none>
Events:               <none>
```

-> default-backend: einige Controller senden Traffic, der nicht durch eine Regel erfasst wird an den Service default-http-backend.

Der Service müsste nun unter *http://alpaca.example.com.* erreichbar sein. 

### Ingress Using Paths

The next interesting scenario is to direct traffic based on not just the hostname, but also the path in the HTTP request.

```
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
 name: path-ingress
spec:
 rules:
 - host: bandicoot.example.com
   http:
     paths:
     - pathType: Prefix
       path: "/"
       backend:
         service:
           name: bandicoot
           port:
             number: 8080
     - pathType: Prefix
       path: "/a/"
       backend:
         service:
           name: alpaca
           port:
             number: 8080
``` 

Aufrufe nach / werden an bandicoot.example.com gesendet, Aufrufe nach /a/ werden nach alpaca gesendet. 

As requests get proxied to the upstream service, the path remains unmodified. That means a request to bandicoot.example.com/a/ shows up to the upstream server that is configured for that request hostname and path. The upstream service needs to be ready to serve traffic on that subpath. In this case, kuard has special code for testing, where it responds on the root path (/) along with a predefined set of subpaths (/a/, /b/, and /c/).

```
kubectl describe ingress path-ingress
Name:             path-ingress
Labels:           <none>
Namespace:        default
Address:
Ingress Class:    <none>
Default backend:  <default>
Rules:
  Host                   Path  Backends
  ----                   ----  --------
  bandicoot.example.com
                         /     bandicoot:8080 (10.244.0.29:8080,10.244.0.37:8080,10.244.0.186:8080)
                         /a/   alpaca:8080 (10.244.0.154:8080,10.244.0.233:8080,10.244.0.114:8080)
Annotations:             <none>
Events:                  <none>
```

### Advanced Ingress Topics
Many of the extended features are exposed via annotations on the Ingress object. Be careful; these annotations can be hard to validate and are easy to get wrong. Many of these annotations apply to the entire Ingress object and so can be more general than you might like. To scope the annotations down, you can always split a single Ingress object into multiple Ingress objects. The Ingress controller should read them and merge them together.

### TLS

1. users need to specify a Secret with their TLS certificate and keys—something like what is outlined in Example 8-4. You can also create a Secret imperatively with kubectl create secret tls \<secret-name\> --cert \<certificate-pem-file\> --key \<private-key-pem-file\>.

2. tls-secret.yaml

```
apiVersion: v1
kind: Secret
metadata:
  creationTimestamp: null
  name: tls-secret-name
type: kubernetes.io/tls
data:
  tls.crt: <base64 encoded certificate>
  tls.key: <base64 encoded private key>
```

3. Once you have the certificate uploaded, you can reference it in an Ingress object. This specifies a list of certificates along with the hostnames that those certificates should be used for:

```
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
 name: tls-ingress
spec:
 tls:
 - hosts:
 - alpaca.example.com
 secretName: tls-secret-name
 rules:
  - host: alpaca.example.com
    http:
      paths:
    - backend:
        serviceName: alpaca
        servicePort: 8080
```

Lets Encrypt Certificates: is API-driven, it is possible to set up a Kubernetes cluster that automatically fetches and installs TLS certificates for you. The missing piece is an open source project called cert-manager created by Jetstack, a UK startup, onboarded to the CNCF. The cert-manager.io website or GitHub repository has details on how to install cert-manager and get started.

### Alternate Ingress Implementation

The most popular generic Ingress controller is probably the open source NGINX Ingress controller.

Emissary and Gloo are two other Envoy-based Ingress controllers that are focused on being API gateways.

Traefik is a reverse proxy implemented in Go that also can function as an Ingress controller. It has a set of features and dashboards that are very developer-friendly.


-> Wichtige URL für weitere Informationen: https://gateway-api.sigs.k8s.io/

# Replica Sets

A ReplicaSet acts as a cluster-wide Pod manager, ensuring that the right types and numbers of Pods are running at all times.


Redundancy - Failure toleration by running multiple instances.

Scale - Higher request-processing capacity by running multiple instances.

Sharding - Different replicas can handle different parts of a computation in parallel


The actual act of managing the replicated Pods is an example of a reconciliation loop. Such loops are fundamental to most of the design and implementation of Kubernetes.

## Reconciliation Loops

The central concept behind a reconciliation loop is the notion of desired state versus observed or current state.

## Relating Pods and ReplicaSets

the relationship between ReplicaSets and Pods is loosely coupled. Though ReplicaSets create and manage Pods, they do not own the Pods they create. ReplicaSets use label queries to identify the set of Pods they should be managing.

ReplicaSets are designed for stateless (or nearly stateless) services. 


## ReplicaSet Spec

```
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  labels:
    app: kuard
    version: "2"
  name: kuard
spec:
  replicas: 1
  selector:
    matchLabels:
      app: kuard
      version: "2"
  template:
    metadata:
      labels:
        app: kuard
        version: "2"
    spec:
      containers:
        - name: kuard
          image: "docker.io/ksaito1125/kuard"
```

-> erstellt einen Pod

Beschreibung des Replica Set über: `kubectl describe rs kuard`
Welches Replicaset steuert den Pod?: `kubectl get pods kuard-vmq5s -o=jsonpath='{.metadata.ownerReferences[0].name}'`

## Scaling and Autoscaling

scale ReplicaSets up or down by updating the spec.replicas

AdHoc: `kubectl scale replicasets kuard --replicas=4`

oder in yaml File um Persistenz sicherzustellen. 

Autoscaling:
-------------

Scale in response to custom application metrics (Horizontal Pod Autoscaling, HPA)

Autoscaling requires the presence of the metrics-server. Most installations of Kubernetes include metrics-server by default. 

Bsp.: für CPU basiertes Autoscaling `kubectl autoscale rs kuard --min=2 --max=5 --cpu='80%'`

Überprüfung: 

```
 kubectl get hpa
NAME    REFERENCE          TARGETS              MINPODS   MAXPODS   REPLICAS   AGE
kuard   ReplicaSet/kuard   cpu: <unknown>/80%   2         5         3          2m8s
```

Because of the decoupled nature of Kubernetes, there is no direct link between the HPA and the ReplicaSet. While this is great for modularity and composition, it also enables some antipatterns. In particular, it’s a bad idea to combine autoscaling with imperative or declarative management of the number of replicas. If both you and an autoscaler are attempting to modify the number of replicas, it’s highly likely that you will clash, resulting in unexpected behavior.

Get possible metrics: `kubectl get --raw "/apis/metrics.k8s.io/v1beta1/namespaces/default/pods"`

## Deleting ReplicaSets and Autoscale

ReplicaSet and PODS!: `kubectl delete rs kuard`
Only ReplicaSet: `kubectl delete rs kuard --cascade=orphan`

Autoscale: `kubectl delete hpa kuard`


# Deployments

Deployments enable you to easily move from one version of your code to the next.
ReplicaSets manage Pods, Deployments manage ReplicaSets. 

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: kuard
  labels:
    run: kuard
spec:
  replicas: 1
  selector:
    matchLabels:
      run: kuard
  template:
    metadata:
      labels:
        run: kuard
    spec:
      containers:
      - name: kuard
        image: docker.io/ksaito1125/kuard
```

Get Label of Deployment: `kubectl get deployments kuard -o jsonpath --template '{.spec.selector.matchLabels}'`

Get ReplicaSet: `kubectl get replicasets --selector=run=kuard`  - hier is run=kuard der Output vom Deployment, ein selector=kuard findet nichts.

Resize Deployment: `kubectl scale deployments kuard --replicas=2`  - ReplicaSet wird angepasst

!! Resize ReplicaSet: `kubectl scale rs kuard-7fb4557664 --replicas=1` -> führt nicht zu dem gewünschten Ergebnis. RS bleibt bei 2

Deployment in Datei speichern: `kubectl get deployments kuard -o yaml > kuard-deployment.yaml`. Bei der weiteren Verwendung aber darauf achten, dass die Read-Only Bereiche gelöscht sind. Das Minifest soll nur enthalten: apiVersion, kind, metadata (name, labels, optional annotations you set yourself), spec



Replace a Deployment: `kubectl replace -f kuard-deployment.yaml --save-config`

You also need to run kubectl replace --save-config. This adds an annotation so that, when applying changes in the future, kubectl will know what the last applied configuration was for smarter merging of configs. If you always use kubectl apply, this step is only required after the first time you create a Deployment using kubectl create -f.

Get Deployment Information 

```
kubectl describe deployments kuard
Name:                   kuard
Namespace:              default
CreationTimestamp:      Tue, 25 Nov 2025 09:53:37 +0100
Labels:                 run=kuard
Annotations:            deployment.kubernetes.io/revision: 1
Selector:               run=kuard
Replicas:               2 desired | 2 updated | 2 total | 2 available | 0 unavailable
StrategyType:           RollingUpdate
MinReadySeconds:        0
RollingUpdateStrategy:  25% max unavailable, 25% max surge
Pod Template:
  Labels:  run=kuard
  Containers:
   kuard:
    Image:         docker.io/ksaito1125/kuard
    Port:          <none>
    Host Port:     <none>
    Environment:   <none>
    Mounts:        <none>
  Volumes:         <none>
  Node-Selectors:  <none>
  Tolerations:     <none>
Conditions:
  Type           Status  Reason
  ----           ------  ------
  Progressing    True    NewReplicaSetAvailable
  Available      True    MinimumReplicasAvailable
OldReplicaSets:  <none>                                      <--- bei Update wird das gesetzt, danach wieder auf none gesetzt
NewReplicaSet:   kuard-7fb4557664 (2/2 replicas created)     
Events:
  Type    Reason             Age                    From                   Message
  ----    ------             ----                   ----                   -------
  Normal  ScalingReplicaSet  14m                    deployment-controller  Scaled up replica set kuard-7fb4557664 from 0 to 1
  Normal  ScalingReplicaSet  7m16s (x2 over 8m36s)  deployment-controller  Scaled up replica set kuard-7fb4557664 from 1 to 2
```

## Updating Deployments

The two most common operations on a Deployment are scaling and application updates.

Verfahren wie üblich. YAML - File anpassen, bei Scaling die Anzahl der Replicas, bei Update i.d.R. ein neues Image. 

Bei einem Annotate soll man auch ein paar Informationen über das Update mit eintragen: 

```
...
spec:
 ...
 template:
   metadata:
     annotations:
       kubernetes.io/change-cause: "Update to green kuard"
 ...
```

Make sure you add this annotation to the template and not the Deployment itself, since the kubectl apply command uses this field in the Deployment object. Also, do not update the changecause annotation when doing simple scaling operations. A modification of change-cause is a significant change to the template and will trigger a new rollout.

```
kubectl apply -f kuard-deployment.yaml
kubectl rollout status deployments kuard
kubectl get replicasets -o wide
kubectl rollout pause deployments kuard
kubectl rollout resume deployments kuard
kubectl rollout history deployment kuard                      - history of deployments
kubectl rollout history deployment kuard --revision=2         - info about deployment nr. 2
kubectl rollout undo deployments kuard                        - undo last deployment
```

By default, the last 10 revisions of a Deployment are kept attached to the Deployment object itself. It is recommended that if you have Deployments that you expect to keep around for a long time, you set a maximum history size for the Deployment revision history. For example, if you do a daily update, you may limit your revision history to 14, to keep a maximum of two weeks’ worth of revisions (if you don’t expect to need
to roll back beyond two weeks).

To accomplish this, use the revisionHistoryLimit property in the Deployment specification:
```
...
spec:
 # We do daily rollouts, limit the revision history to two weeks of
 # releases as we don't expect to roll back beyond that.
 revisionHistoryLimit: 14
...
```

## Deployment Strategies

Recreate or RollingUpdate. 

Recreate strategy: Updates the ReplicaSet and terminates all of the Pods associated with the Deployment and recreates all Pods with new image.
-> Downtime

RollingUpdate: Importantly, this means that for a while, both the new and the old version of your service will be receiving requests and serving traffic. This has important implications for how you build your software.

### Configuring a rolling update

There are two parameters you can use to tune the rolling update behavior: maxUnavailable and maxSurge.

maxUnavailable: max. number of Pods that can be unavailable during a rolloing update (count or percent).

maxSurge: how many extra resources can be created or achive a rollout. To illustrate how this works, imagine a service with 10 replicas. We set
maxUnavailable to 0 and maxSurge to 20%. The first thing the rollout will do is scale the new ReplicaSet up by 2 replicas, for a total of 12 (120%) in the service. It will then scale the old ReplicaSet down to 8 replicas, for a total of 10 (8 old, 2 new) in the service. This process proceeds until the rollout is complete. At any time, the capacity of the service is guaranteed to be at least 100% and the maximum extra resources used for the rollout are limited to an additional 20% of all resources.


### Ensure Service Health

For Deployments, this time to wait is defined by the minReadySeconds parameter:

```
...
spec:
 minReadySeconds: 60
...
```

Setting minReadySeconds to 60 indicates that the Deployment must wait for 60 seconds after seeing a Pod become healthy before moving on to updating the next Pod.

In order to set the timeout period, you will use the Deployment parameter progress DeadlineSeconds:

```
...
spec:
 progressDeadlineSeconds: 600
...
```

This example sets the progress deadline to 10 minutes. If any particular stage in the rollout fails to progress in 10 minutes, then the Deployment is marked as failed, and all attempts to move the Deployment forward are halted.

## Deleting a Deployment

`kubectl delete deployments kuard`
`kubectl delete -f kuard-deployment.yaml` - delete also rs
`kubectl delete -f kuard-deployment.yaml --cascade=orphan`   - delete only deployment


# Daemon Sets

A DaemonSet ensures that a copy of a Pod is running across a set of nodes in a Kubernetes cluster. DaemonSets are used to deploy system daemons such as log col‐
lectors and monitoring agents, which typically must run on every node. DaemonSets share similar functionality with ReplicaSets; both create Pods that are expected to be
long-running services and ensure that the desired state and the observed state of the cluster match.

* ReplicaSets should be used when your application is completely decoupled from the node and you can run multiple copies on a given node without special consideration. 
* DaemonSets should be used when a single copy of your application must run on all or a subset of the nodes in the cluster

You should generally not use scheduling restrictions or other parameters to ensure that Pods do not colocate on the same node. If you find yourself wanting a single Pod
per node, then a DaemonSet is the correct Kubernetes resource to use. Likewise, if you find yourself building a homogeneous replicated service to serve user traffic, then a ReplicaSet is probably the right Kubernetes resource to use.

By default, a DaemonSet will create a copy of a Pod on every node unless a node selector is used, which will limit eligible nodes to those with a matching set of labels.

```
apiVersion: apps/v1
kind: DaemonSet   <-- Daemon Set
metadata:
  name: fluentd
  labels:
    app: fluentd
spec:
  selector:
    matchLabels:
      app: fluentd
  template:
    metadata:
      labels:
        app: fluentd
    spec:
      containers:
      - name: fluentd
        image: fluent/fluentd:v0.14.10
        resources:
          limits:
            memory: 200Mi
          requests:
            cpu: 100m
            memory: 200Mi
        volumeMounts:
        - name: varlog
          mountPath: /var/log
        - name: varlibdockercontainers
          mountPath: /var/lib/docker/containers
          readOnly: true
      terminationGracePeriodSeconds: 30
      volumes:
      - name: varlog
        hostPath:
          path: /var/log
      - name: varlibdockercontainers
        hostPath:
          path: /var/lib/docker/containers
```

With the fluentd DaemonSet in place, adding a new node to the cluster will result in a fluentd Pod being deployed to that node automatically.

## Limiting DaemonSets to Specific Nodes

* Adding Labels to Nodes

```yaml
# Add Label to node
kubectl label nodes k0-default-pool-35609c18-z7tb ssd=true    
kubectl get nodes --selector ssd=true

# pod file
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: nginx-fast-storage
  labels:
    app: nginx
    ssd: "true"
spec:
  selector:
    matchLabels:
      app: nginx
      ssd: "true"
  template:
    metadata:
      labels:
        app: nginx
        ssd: "true"
    spec:
      nodeSelector:
        ssd: "true"    <-- wähle jene Nodes mit dem Label
      containers:
      - name: nginx
        image: nginx:1.10.0
```

**Removing labels from a node that are required by a DaemonSet’s node selector will cause the Pod being managed by that DaemonSet to be removed from the node.**


## Update a Daemon-Set

DaemonSets can be rolled out using the same RollingUpdate strategy that Deployments use.

There are two parameters that control the rolling update of a DaemonSet:

* *spec.minReadySeconds*  Determines how long a Pod must be “ready” before the rolling update proceeds to upgrade subsequent Pods

* *spec.updateStrategy.rollingUpdate.maxUnavailable*   Indicates how many Pods may be simultaneously updated by the rolling update

# Jobs

A Job creates Pods that run until successful termination (for instance, exit with 0). In contrast, a regular Pod will continually restart regardless of its exit code. Jobs are
useful for things you only want to do once, such as database migrations or batch jobs.

The Job object is responsible for creating and managing Pods defined in a template in the job specification. These Pods generally run until successful completion. The Job
object coordinates running a number of Pods in parallel.

## Job patterns
By default, each job runs a single Pod once until successful termination. This job pattern is defined by two primary attributes of a job: the **number of job completions** and the **number of Pods to run in parallel**.

### One Shot

One-shot jobs provide a way to run a single Pod once until successful termination.

```
apiVersion: batch/v1
kind: Job
metadata:
  name: oneshot
spec:
  template:
    spec:
      containers:
      - name: kuard
        image: gcr.io/kuar-demo/kuard-amd64:blue
        imagePullPolicy: Always
        command:
        - "/kuard"
        args:
        - "--keygen-enable"
        - "--keygen-exit-on-complete"
        - "--keygen-num-to-gen=10"
      restartPolicy: OnFailure

kubectl delete pod oneshot
```

Because jobs have a finite beginning and ending, users often create many of them. This makes picking unique labels more difficult and more critical. For this reason, the Job object will automatically pick a unique label and use it to identify the Pods it creates. In advanced scenarios (such as swapping out a running job without killing the
Pods it is managing), users can choose to turn off this automatic behavior and manually specify labels and selectors.

### Parallelism

10 Completions, parallel 5 Jobs

```
apiVersion: batch/v1
kind: Job
metadata:
  name: parallel
  labels:
    chapter: jobs
spec:
  parallelism: 5
  completions: 10
  template:
    metadata:
      labels:
        chapter: jobs
    spec:
      containers:
      - name: kuard
        image: docker.io/ksaito1125/kuard
        imagePullPolicy: Always
        command:
        - "/kuard"
        args:
        - "--keygen-enable"
        - "--keygen-exit-on-complete"
        - "--keygen-num-to-gen=10"
      restartPolicy: OnFailure
```

`kubectl delete job parallel`

### Workqueues

Some task creats a number of work items and publishes them to a work queue. A worker job can be run to process each work item until the work queue is empty.

#### Starting a workqueue

We start by launching a centralized work queue service. kuard has a simple memorybased work queue system built in.

```
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: queue
  labels:
    app: work-queue
    component: queue
    chapter: jobs
spec:
  replicas: 1
  selector:
    matchLabels:
      app: work-queue
      component: queue
      chapter: jobs
  template:
    metadata:
      labels:
        app: work-queue
        component: queue
        chapter: jobs
    spec:
      containers:
      - name: queue
        image: "docker.io/ksaito1125/kuard"
        imagePullPolicy: Always
``` 

Create a service

```
apiVersion: v1
kind: Service
metadata:
  name: queue
  labels:
    app: work-queue
    component: queue
    chapter: jobs
spec:
  ports:
    - port: 8080
      protocol: TCP
      targetPort: 8080
  selector:
    app: work-queue
    component: queue
    chapter: jobs
```
 
```
kubectl get services
NAME                  TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)    AGE
kubernetes            ClusterIP   10.0.0.1       <none>        443/TCP    22d
queue                 ClusterIP   10.0.16.181    <none>        8080/TCP   10s
```

#### Loading the queue

```
# Create a work queue called 'keygen'
curl -X PUT localhost:8080/memq/server/queues/keygen

# Create 100 work items and load up the queue.
for i in work-item-{0..99}; do
 curl -X POST localhost:8080/memq/server/queues/keygen/enqueue -d "$i"
done


Powershell:
# Create the queue
Invoke-RestMethod -Uri "http://localhost:8080/memq/server/queues/keygen" -Method Put

# Create 100 work items and enqueue them
for ($i = 0; $i -lt 100; $i++) {
    $workItem = "work-item-$i"
    Invoke-RestMethod -Uri "http://localhost:8080/memq/server/queues/keygen/enqueue" `
                      -Method Post `
                      -Body $workItem
}

```

#### create a consumer job

```
apiVersion: batch/v1
kind: Job
metadata:
  name: consumers
  labels:
    app: message-queue
    component: consumer
    chapter: jobs
spec:
  parallelism: 5
  template:
    metadata:
      labels:
        app: message-queue
        component: consumer
        chapter: jobs
    spec:
      containers:
      - name: worker
        image: "docker.io/ksaito1125/kuard"
        imagePullPolicy: Always
        command:
        - "/kuard"
        args:
        - "--keygen-enable"
        - "--keygen-exit-on-complete"
        - "--keygen-memq-server=http://queue:8080/memq/server"
        - "--keygen-memq-queue=keygen"
      restartPolicy: OnFailure
```

`kubectl apply -f job-consumers.yaml`

Note there are five Pods running in parallel. These Pods will continue to run until the work queue is empty. You can watch as it happens in the UI on the work queue server. As the queue empties, the consumer Pods will exit cleanly and the consumers job will be considered complete.

### Cronjobs

Sometimes you want to schedule a job to be run at a certain interval. To achieve this, you can declare a CronJob in Kubernetes, which is responsible for creating a new Job object at a particular interval.

```
apiVersion: batch/v1
kind: CronJob
metadata:
  name: example-cron
spec:
  # Run every fifth hour
  schedule: "0 */5 * * *"   <- Standard cron format
  jobTemplate:
    spec:
      template:
        spec:
          containers:
          - name: batch-job
            image: my-batch-image
          restartPolicy: OnFailure
```

```
kubectl get cronjob.batch
NAME           SCHEDULE      TIMEZONE   SUSPEND   ACTIVE   LAST SCHEDULE   AGE
example-cron   0 */5 * * *   <none>     False     0        <none>          3m17s

kubectl describe cronjob.batch/example-cron
```

# ConfigMap & Secrets

ConfigMaps are used to provide configuration information for workloads. Secrets are similar to ConfigMaps but focus on making sensitive information available to the workload.

## ConfigMaps

ConfigMap is a set of variables that can be used when defining the environment or command line for your containers. The key thing to note is that
the ConfigMap is combined with the Pod right before it is run. This means that the container image and the Pod definition can be reused by many workloads just by
changing the ConfigMap that is used.

### Creating ConfigMaps

Parameterdatei, in der Form
```
parameter1=value1
parameter2=value"
```

`kubectl create configmap my-config --from-file=my-config.txt --from-literal=extra-param=extra-value --from-literal=another-param=another-value`

```
kubectl get configmaps my-config -o yaml
apiVersion: v1
data:
  another-param: another-value
  extra-param: extra-value
  my-config.txt: "# This is a sample config file that I might use to configure an
    application\r\nparameter1 = value1\r\nparameter2 = value2\r\n"
kind: ConfigMap
metadata:
  creationTimestamp: "2025-12-09T15:15:03Z"
  name: my-config
  namespace: default
  resourceVersion: "3064113"
  uid: b993a192-6aed-464e-a1fd-24c3b038c899
```

### Using a ConfigMap

1. Filesystem -> mount in container
2. Environment Variable 
3. Command-Line argument


```
apiVersion: v1
kind: Pod
metadata:
  name: kuard-config
spec:
  containers:
  - name: test-container
    image: docker.io/ksaito1125/kuard
    imagePullPolicy: Always
    command:
    - "/kuard"
    - "$(EXTRA_PARAM)"
    env:
      # An example of an environment variable used inside the container
      - name: ANOTHER_PARAM
        valueFrom:
          configMapKeyRef:
            name: my-config
            key: another-param
      # An example of an environment variable passed to the command to start
      # the container (above).
      - name: EXTRA_PARAM
        valueFrom:
          configMapKeyRef:
            name: my-config
            key: extra-param
    volumeMounts:
      # Mounting the ConfigMap as a set of files
      - name: config-volume
        mountPath: /config
  volumes:
  - name: config-volume
    configMap:
      name: my-config
  restartPolicy: Never
```

For the filesystem method, we create a new volume inside the Pod and give it the name config-volume.

Environment variables are specified with a special valueFrom member.

Command-line arguments build on environment variables.

```
kubectl apply -f kuard-config.yaml
kubectl port-forward kuard-config 8080
kubectl exec -it kuard-config -- sh
```

In the pod in dir /config is also a config-map stored

## Secrets

Secrets are exposed to Pods via explicit declaration in Pod manifests and the Kubernetes API.

*By default, Kubernetes Secrets are stored in plain text in the etcd storage for the cluster. Depending on your requirements, this may not be sufficient security for you. In particular, anyone who has cluster administration rights in your cluster will be able to read all of the Secrets in the cluster.*
*In recent versions of Kubernetes, support has been added for encrypting the Secrets with a user-supplied key, generally integrated into a cloud key store. Additionally, most cloud key stores have integration with Kubernetes Secrets Store CSI Driver volumes, enabling you to skip Kubernetes Secrets entirely and rely exclusively on the cloud provider’s key store. All of these options should provide you with sufficient tools to craft a security profile that suits your needs.*

### Creating Secrets

*The kuard container image does not bundle a TLS certificate or key. This allows the kuard container to remain portable across environments and distributable through public Docker repositories.*

1. Get certificates
``` 
$ curl -o kuard.crt https://storage.googleapis.com/kuar-demo/kuard.crt
$ curl -o kuard.key https://storage.googleapis.com/kuar-demo/kuard.key
```
2. Create a secret 
`kubectl create secret generic kuard-tls --from-file=kuard.crt --from-file=kuard.key`

` kubectl describe secrets kuard-tls`

### Consume Secrets

Instead of accessing Secrets through the API server, we can use a Secrets volume. Secret data can be exposed to Pods using the Secrets volume type. Secrets volumes are managed by the kubelet and are created at Pod creation time. Secrets are stored on tmpfs volumes (aka RAM disks), and as such are not written to disk on nodes.

Mounting the kuard-tls Secrets volume to /tls results in the following files:

/tls/kuard.crt
/tls/kuard.key

```
apiVersion: v1
kind: Pod
metadata:
  name: kuard-tls
spec:
  containers:
  - name: kuard-tls
    image: docker.io/ksaito1125/kuard
    imagePullPolicy: Always
    volumeMounts:
    - name: tls-certs
      mountPath: "/tls"
      readOnly: true
  volumes:
  - name: tls-certs
    secret:
      secretName: kuard-tls
```

```
kubectl apply -f kuard-secret.yml
kubectl port-forward kuard-tls 8443:8443
https://localhost:8443
```

### Private Container Registry

A special use case for Secrets is to store access credentials for private container registries.Kubernetes supports using images stored on private registries, but access to those images requires credentials.

Image pull Secrets leverage the Secrets API to automate the distribution of private registry credentials. Image pull Secrets are stored just like regular Secrets but are consumed through the spec.imagePullSecrets Pod specification field.

Use kubectl create secret docker-registry to create this special kind of Secret:
```
$ kubectl create secret docker-registry my-image-pull-secret \
 --docker-username=<username> \
 --docker-password=<password> \
 --docker-email=<email-address>
```

Enable access to the private repository by referencing the image pull secret in the Pod manifest file

```
apiVersion: v1
kind: Pod
metadata:
  name: kuard-tls
spec:
  containers:
  - name: kuard-tls
    image: docker.io/ksaito1125/kuard
    imagePullPolicy: Always
    volumeMounts:
    - name: tls-certs
      mountPath: "/tls"
      readOnly: true
  imagePullSecrets:
  - name: my-image-pull-secret
  volumes:
  - name: tls-certs
    secret:
      secretName: kuard-tls
```

If you are repeatedly pulling from the same registry, you can add the Secrets to the default service account associated with each Pod to avoid having to specify the Secrets in every Pod you create.

## Naming Constraints

When selecting a key name, remember that these keys can be exposed to Pods via a volume mount. Pick a name that is going to make sense when specified on a command line or in a config file. Storing a TLS key as key.pem is clearer than tls-key when configuring applications to access Secrets.

## Managing ConfigMaps and Secrets

ConfigMaps and Secrets are managed through the Kubernetes API. The usual create, delete, get, and describe commands work for manipulating these objects.

```
# List
kubectl get secrets
kubectl get configmaps
kubectl describe configmap my-config

# create
kubectl create secret generic ...
kubectl create configmap ...

mit ...
--from-file=filename
--from-file=key=filename (load from the file with the Secret data key explicitly specified.)
--from-file=directory
--from-literal=key=value (use specified key/value pair)

# Update

If you have a manifest for your ConfigMap or Secret, you can just edit it directly and replace it with a new version using kubectl replace -f <filename>. You can
also use kubectl apply -f <filename> if you previously created the resource with kubectl apply.

# recreate & update

kubectl create secret generic kuard-tls --from-file=kuard.crt --from-file=kuard.key --dry-run -o yaml | kubectl replace -f -

This command line first creates a new Secret with the same name as our existing Secret. If we just stopped there, the Kubernetes API server would return an error complaining that we are trying to create a Secret that already exists. Instead, we tell kubectl not to actually send the data to the server but instead to dump the YAML that it would have sent to the API server to stdout. We then pipe that to kubectl replace and use -f - to tell it to read from stdin. In this way, we can update a Secret from files on disk without having to manually base64-encode data.

# edit current version

kubectl edit configmap my-config
```

# Role-Based Access Control for Kubernetes

## RBAC

### Identities in kubernetes

Every request to Kubernetes is associated with some identity. Even a request with no identity is associated with the system:unauthenticated group. Kubernetes makes a
distinction between user identities and service account identities. Service accounts are created and managed by Kubernetes itself and are generally associated with components running inside the cluster. User accounts are all other accounts associated with actual users of the cluster, and often include automation like continuous delivery services that run outside the cluster.

You should always use different identities for different applications in your cluster.

For example, you should have one identity for your production frontends, a different identity for the production backends, and all production identities should be distinct
from development identities. s. You should also have different identities for different clusters. All of these identities should be machine identities that are not shared with
users. You can either use Kubernetes Service Accounts for achieving this, or you can use a Pod identity provider supplied by your identity system; for example, Azure Active Directory supplies an open source identity provider for Pods as do other popular identity providers.

A role is a set of abstract capabilities. 

A role binding is an assignment of a role to one or more identities. 

In Kubernetes, two pairs of related resources represent roles and role bindings. One pair is scoped to a namespace (Role and RoleBinding), while the other pair is scoped to the cluster (ClusterRole and ClusterRoleBinding).

Role resources are namepaced and represent capabilities within that single namespace. You cannot use namespaced roles for nonnamespaced resources

Example Role

```yaml
kind: Role
apiVersion: rbac.authorization.k8s.io/v1
metadata:
 namespace: default
 name: pod-and-services
rules:
- apiGroups: [""]
  resources: ["pods", "services"]
  verbs: ["create", "delete", "get", "list", "patch", "update", "watch"]
```

Example Binding

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
 namespace: default
 name: pods-and-services     
subjects:
- apiGroup: rbac.authorization.k8s.io
 kind: User
 name: alice                 <- user
- apiGroup: rbac.authorization.k8s.io
 kind: Group
 name: mydevs                <- or group
roleRef:
 apiGroup: rbac.authorization.k8s.io
 kind: Role
 name: pod-and-services      <- role
```

Sometimes you need to create a role that applies to the entire cluster, or you want to limit access to cluster-level resources. To achieve this, you use the ClusterRole and
ClusterRoleBinding resources. They are largely identical to their namespaced peers, but are cluster-scoped.


| Verb | HTTP |method Description                                  |
| -----|------|--------------------------------------------------- |
| create| POST| Create a new resource.                             |
| delete | DELETE| Delete an existing resource.                    |
| get | GET |Get a resource.                                       |
| list | GET |List a collection of resources.                      |
| patch | PATCH| Modify an existing resource via a partial change. |
| update | PUT |Modify an existing resource via a complete object. |
| watch | GET |Watch for streaming updates to a resource.          |
| proxy | GET |Connect to resource via a streaming WebSocket proxy |


### Built-in roles

kubectl get clusterroles

* The cluster-admin role provides complete access to the entire cluster.
* The admin role provides complete access to a complete namespace.
* The edit role allows an end user to modify resources in a namespace.
* The view role allows for read-only access to a namespace.

Bindings: `kubectl get clusterrolebindings`

When the Kubernetes API server starts up, it automatically installs a number of default ClusterRoles that are defined in the code of the API server itself. This means that if you modify any built-in cluster role, those modifications are transient. Whenever the API server is restarted (e.g., for an upgrade), your changes will be overwritten.

To prevent this from happening, before you make any other modifications, you need to add the rbac.authorization.kubernetes.io/autoupdate annotation with a value of false to the built-in ClusterRole resource. If this annotation is set to false, the API server will not overwrite the modified ClusterRole resource.

**By default, the Kubernetes API server installs a cluster role that allows system:unauthenticated users access to the API server’s API discovery endpoint. For any cluster exposed to a hostile environment (e.g., the public internet) this is a bad idea, and there has been at least one serious security vulnerability via this exposure. If you are running a Kubernetes service on the public internet or an other hostile environment, you should ensure that the --anonymous-auth=false flag is set on your API server.**

## Managing RBACS

can-i command for kubectl. This tool is used for testing whether a specific user can perform a specific action. You can use can-i to validate configuration settings as you configure your cluster, or you can ask users to use the tool to validate their access when filing errors or bug reports.

```
kubectl auth can-i create pods
kubectl auth can-i get pods --subresource=logs
```
RBAC resources are modeled using YAML. Given this text-based representation, it makes sense to store these resources in version control, which allows for accountability, auditability, and rollback.

`kubectl auth reconcile -f some-rbac-config.yaml`

**Aggregate**

Sometimes you want to be able to define roles that are combinations of other roles. Kubernetes RBAC supports the usage of an aggregation rule to combine multiple roles in a new role. This new role combines all of the capabilities of all of the aggregate roles, and any changes to any of the constituent subroles will automatically be propogated back into the aggregate role. As with other aggregations or groupings in Kubernetes, the ClusterRoles to be aggregated are specified using label selectors. In this particular case, the aggregationRule field in the ClusterRole resource contains a clusterRoleSelector field, which in turn is a label selector. All ClusterRole resources that match this selector are dynamically aggregated into the rules array in the aggregate ClusterRole resource.

example: the built-in edit role looks like this:

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
 name: edit
 ...
aggregationRule:
 clusterRoleSelectors:
 - matchLabels:
 rbac.authorization.k8s.io/aggregate-to-edit: "true"
...
``` 

This means that the edit role is defined to be the aggregate of all ClusterRole objects that have a label of rbac.authorization.k8s.io/aggregate-to-edit set to true.

**Bindings**

When you bind a group to a Role or ClusterRole, anyone who is a member of that group gains access to the resources and verbs defined by that role.

To bind a group to a ClusterRole, use a Group kind for the subject in the binding:

```yaml
...
subjects:
- apiGroup: rbac.authorization.k8s.io
 kind: Group
 name: my-great-groups-name
...
```


# Service Mesh

There are three general capabilities provided by most service mesh implementations: network encryption and authorization, traffic shaping, and observability

## Encryption and Authentication with Mutal TLS

Encryption of network traffic between Pods is a key component to security in a microservice architecture. Encryptions provided by Mutual Transport Layer Security,
or mTLS, is one of the most popular use cases for a service mesh. installing a service mesh on your Kubernetes cluster automatically provides encryption to network traffic between every Pod in the cluster. The service mesh adds a sidecar container to every Pod, which transparently intercepts all network communication. 

## Traffic shaping

In a dog-fooding model, you may run version Y of your service for a day to a week (or longer) for a subset of users before you roll it out broadly to your full set of users.
Such experiments require the ability to do traffic shaping, or routing of requests to different service implementations based on the characteristics of the request.











































