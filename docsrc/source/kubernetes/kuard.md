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

Kubernetes calls its HTTP-based load-balancing system **Ingress**. Ingress is aKubernetes-native way to implement the “virtual hosting” pattern we just discussed. One of the more complex aspects of the pattern is that the user has to manage the load balancer configuration file. In a dynamic environment and as the set of virtual hosts expands, this can be very complex. The Kubernetes Ingress system works to simplify this by (a) standardizing that configuration, (b) moving it to a standard Kubernetes object, and (c) merging multiple Ingress objects into a single config for the load balancer.

The Ingress controller is a software system made up of two parts. 
* The first is the Ingress proxy, which is exposed outside the cluster using a service of type: LoadBalancer. This proxy sends requests to “upstream” servers. 
* The other component is the Ingress reconciler, or operator. The Ingress operator is responsible for reading and monitoring Ingress objects in the Kubernetes API and reconfiguring the Ingress proxy to route traffic as specified in the Ingress resource. 

## Ingress Spec

The Spec is split into a common resource specification and a controller implementation. There is no "standard" Ingress controller that is built into Kubernetes, so the user must install one of many optional implementations.

Users can create and modify Ingress objects just like every other object. But, by default, there is no code running to actually act on those objects.



