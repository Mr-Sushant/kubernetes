# Kubernetes
It is an open source system for automating deployment, scaling and management of containerized applications.
Link - (https://kubernetes.io)

## Need for Kubernetes
1. Package an app and let something else manage it for us.
2. No worry about container management
3. Eliminate single points of failure
4. Scale containers
5. Update containers without downtime
6. have robust and persistent storage

## Key Container Benefits
1. Accelerate developer onboarding
2. Eliminate app conflicts
3. Environment consistency
4. Ship software faster

## Key Kubernetes Benefits
1. Orchestrate containers
2. Zero downtime deployment
3. Self Healing
4. Scale containers horizontally

## Kubectl commands
1. kubectl version - check kubernetes version
2. kubectl cluster-info - view cluster information
3. kubectl get all - retrieve information about kubernetes pods, deployments, services, etc.
4. kubectl run \[container-name\] --image=\[image-name\] - create a deployment for a pod
5. kubectl port-forward \[pod\] \[ports\] - forward a port to allow external access.
6. kubectl expose ... - expose a port for a deployment/Pod
7. kubectl create \[resource\] - create a resource
8. kubectl apply \[resource\] - create/modify a resource.

## Set alias for kubectl
### Powershell
Set-Alias -Name k -Value kubectl

### linux/Mac
alias k = "kubectl"

## Steps to setup Web UI Dashboard
1. k apply -f https://raw.githubusercontent.com/kubernetes/dashboard/v2.7.0/aio/deploy/recommended.yaml
2. k apply -f dashboard.adminuser.yml (first create this file)
3. k create token admin-user -n kube-system
4. k proxy
5. Visit the URL - http://localhost:8001/api/v1/namespaces/kubernetes-dashboard/services/https:kubernetes-dashboard:/proxy/
6. Paste the generated token

![Dashboard](/pics/dashboard.png "Kubernetes Web UI Dashboard")


## POD
A pod is the basic execution unit of a Kubernetes application - the smallest and simplest unit in the kubernetes object model that we create or deploy.
Pod acts as an environment for a container
Pod IP, memory, volumes, etc can be shared across containers.
We can scale horizontally by adding Pod replicas.
Pods live and die but never come back to life.

- Pod containers share the same network namespace (IP/PORT)
- Pod containers share the same loopback network interface (localhost)
- container processes need to bind to different ports within a pod.
- Ports can be reused by containers in separate pods.
- Pods do not span over multiple nodes

![POD](/pics/pod.png)

### Deleting a Pod
kubectl delete pod \[pod-name\]
`This will cause the pod to be recreated`

kubectl delete deployment \[deployment-name\]
`this will delete the deployment that manages the pod`

## YAML
- composed of maps and lists
- indentation matters
- always use spaces for indent
- Maps:
    - name: value pairs
    - Maps can contain other maps for more complex data structures
- Lists:
    - sequence of items
    - multiple maps can be defined in a list

### Defining a Pod with YAML
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-nginx
spec:
  containers:
  - name: my-nginx
    image: nginx:alpine
```

To create a pod with the above configuration.
```
kubectl create -f file.pod.yml
or
kubectl create -f file.pod.yml --dry-run --validate=true
```
Validate=true is a default value and throws error if yaml is not correct.
--dry-run would not directly effect the cluster instead it will show us a preview how it will look.

create will throw error if the pod is already present.
Instead we can use *kubectl apply -f file.pod.yml*. This will create or update the pod.

```
kubectl create -f file.pod.yml --save-config
```
this will store current properties in resource's annotations (inside metadata)

```
kubectl describe \[pod-name\]
```

## Pod Health
Kubernetes relies on Probes to determine the health of a pod container.
A probe is a diagnostic performed periodically by the kubelet on a container.

### Types of Probes
1. Liveness Probe
2. Readiness Probe

Liveness Probe - determine if a pod is healthy and running as expected.
Readiness Probe - determine if a pod should receive requests.

Failed Pod containers are recreated by default. (restartPolicy defaults to always).

### Probe Types
1. ExecAction - executes an action inside the container.
2. TCPSocketAction - TCP check against the container's IP address on a specified port.
3. HTTPGetAction - HTTP GET request against container.

Probes can have following result values:
- Success
- Failure
- Unknown

```yaml
apiVersion: v1
kind: Pod
...

spec:
    containers:
    -   name: my-nginx
        image: nginx-alpine
        readinessProbe:              # READINESS PROBE
            httpGet:
                path: /index.html
                port: 80
            initialDelaySeconds: 2   # wait 2 seconds
            periodSeconds: 5         # check every 5 seconds

        livenessProbe:               # LIVENESS PROBE
            httpGet:
                path: /index.html
                port: 80
            initialDelaySeconds: 15  # wait 15 seconds
            timeout: 2               # timeout after 2 seconds
            periodSeconds: 5         # check every 5 seconds
            failureThreshold: 1      # allow 1 failure before failing pod
```

## Deployment

### Replica set
A replica set is a declarative way to manage Pods.
A deployment is a declarative way to manage Pods using a ReplicaSet.

Since a Pod cannot be re-created, Deployments and Replicaset ensure pods stay running and can be used to scale Pods.

Replica Set act as a Pod controller:
- Self healing mechanism
- ensure requested number of pods are available
- provide fault tolerance
- used to scale pods
- relies on a pod template
- no need to create pods directly
- used by deployments

### Deployment
A deployment manages Pods:
- pods are managed using ReplicaSet
- scales replicaset, which scales pods
- supports zero-downtime updates by creating and destroying replicaSets
- provides rollback functionality
- creates a unique label that is assigned to the replicaSet and generated Pods
- YAML is very similar to ReplicaSet

![Deployment](/pics/deployment.png)

Show Deployments with all the labels available
```
kubectl get deployment --show-labels
```

Find the deployments with specified labels
```
kubectl get deployment -l app=nginx
```

Delete Deployment
```
kubectl delete deployment \[deployment-name\]
```

Scale the deployment to 5 pods
```
kubectl scale deployment \[deployment-name\] --replicas=5
```

Scale the deployment to 5 pods using YAML file
```
kubectl scale -f \[file-name\] --replicas=5
```

## Deployment Options
One of strengths of K8s is ZERO DOWNTIME DEPLOYMENTS
There are several options available:
1. Rolling updates
2. Blue-Green updates
3. Canary deployments
4. Rollbacks

This happens automatically when we do kubectl apply command.

## Services
A service provides a single point of entry for accessing one or more pods.

### Need for Services ?
Suppose there is a pod with IP 1.1.1.1 and it dies, so its the responsibility of the service to redirect new requests to the new IP of new Pod (2.2.2.2).

- Service abstracts POD IP Address from consumers.
- Load balances between  pods
- relies on labels to associate a service with pod
- node's kube-proxy creates a virtual IP for services
- layer 4 (TCP/UDP over IP)
- Services are not ephemeral
- creates endpoint which sits between a service and a pod

![Services](/pics/service.png)

### Service Types
1. ClusterIP - expose the service on a cluster-internal IP (default). Only pods within the cluster are allowed to talk to the service. It allows Pods to talk to other Pods.
2. NodePort - expose the service on each Node's IP at a static port
3. LoadBalancer - provision an external IP to act as a load balancer for the service
4. ExternalName - Maps a service to a DNS name.

