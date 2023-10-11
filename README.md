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
