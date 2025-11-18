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




