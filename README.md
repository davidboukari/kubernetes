# kubernetes

# Kubernetes commands

## To test commands
* Katacoda: https://www.katacoda.com/courses/kubernetes/playground

## Switch context
* https://github.com/ahmetb/kubectx

## Cluster info
```
$ kubectl cluster-info
Kubernetes master is running at https://192.168.0.155:6443
KubeDNS is running at https://192.168.0.155:6443/api/v1/namespaces/kube-system/services/kube-dns:dns/proxy
```
## Nodes Managements = master & Workers = <none>
```bash
$ kubectl get nodes
NAME                                              STATUS   ROLES    AGE   VERSION
vsphere-management-cluster-control-plane-zffw4    Ready    master   83m   v1.19.1+vmware.2
vsphere-management-cluster-md-0-bcbd45f87-l9xfp   Ready    <none>   76m   v1.19.1+vmware.2
```

## Basic commands for pods

### Get all pods of all namespaces
```
kubectl get pods -A

kubectl get po
kubectl get po/nginx
kubectl describe po/nginx
kubectl exec -it po/nginx -- /bin/bash
kubectl delete vpo/nginx
kubectl port-forward nginx 8080:80

```
###  Kubernetes Services Objects
kube-proxy is available on each pod and it manage the Load Balancing (userspace, iptables, ipvs)
* ClusterIP: Expose into the cluster only
```
cat Service.yaml
apiVersion: v1
kind: Service
metadata:
  name: my-service
spec:
  selector:
    app: MyApp
  ports:
    - protocol: TCP
      port: 80
      targetPort: 9376

cat Pod.yaml      
apiVersion: v1
kind: Pod
metadata:
  name: vote
  labels:
    app: MyApp
spec:
  containers:
  - name: vote
    image: instavote/vote
    ports:
      - containerPort: 80
```

* ClusterIP Port-Forward: Expose into the cluster only

* Cluster-IP Proxy: Expose into the cluster only
* NodePort: Expose to out of the cluster
* LoadBalancer: Integration with the provider
* ExternalName: Associate to DNS name

## Scheduling

### By label

#### Add a label for a node
```
kubectl label  no/my-vsphere-cluster-md-0-686fc88ccd-v5lvb type=bdd
=>
```
#### Select this node for a pod
```
spec:
  containers:
  NodeSelector:
    type: bdd
```

### By nodeAffinity
* In, NotIn, Exists, DoesNotExist, Gt, Lt

#### requiredDuringSchedulingIgnoredDuringExecution (Hard constraint => the pod is not created if it does not match)
```
affinity:
  nodeAffinity:
    requiredDuringSchedulingIgnoredDuringExecution:
      nodeSelectorTerms:
        - matchExpressions:
          - key: "failure-domain.beta.kubernetes.io/zone"
            operator: In
            values: ["us-central1-a"]
```

#### preferredDuringSchedulingIgnoredDuringExecution (Soft constraint => The pod is created but it do the best to try to match)
```
affinity:
  nodeAffinity:
    preferredDuringSchedulingIgnoredDuringExecution:
      nodeSelectorTerms:
        - matchExpressions:
          - key: "failure-domain.beta.kubernetes.io/zone"
            operator: In
            values: ["us-central1-a"]
```
### taint the node as unschedulable by any pods that do not have a toleration for taint with key (taint eviter)
kubectl taint nodes node1 key=value:NoSchedule
```
...
tolerations:
- key: "key"
  operator: "Equal"
  value: "value"
  effect: "NoSchedule"
```
### resource
```
...
resources:
  requests:
    memory: "64Mi"
    cpu: "250"
  limits:
    memory: "128Mi"
    cpu: "500"
```
### podAffinity
```
affinity:
    podAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
      - labelSelector:
          matchExpressions:
          - key: service
            operator: In
            values: [“S1”]
        topologyKey: failure-domain.beta.kubernetes.io/zone
```
### podAntiAffinity
```
affinity:
    podAntiAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
      - labelSelector:
          matchExpressions:
          - key: service
            operator: In
            values: [“S1”]
        topologyKey: failure-domain.beta.kubernetes.io/zone
```
## Deployment Very important replica
* https://cloud.google.com/kubernetes-engine/docs/how-to/horizontal-pod-autoscaling?hl=fr
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  labels:
    app: nginx
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.7.9
        ports:
        - containerPort: 80
```

If the image is update
```
kubectl set image deploy/nginx-deployment nginx=nginx:1.7.9 --record=true
kubectl rollout history deploy/nginx-deployment
REVISION        Change Cause
1               <none>
2               <none>
3               kubectl set image deploy/nginx-deployment nginx=nginx:1.7.9 --record=true
```

kubectl scale deploy/nginx-deployment --replicas=5


### HorizontalPodAutoscaler
* https://kubernetes.io/docs/tasks/run-application/horizontal-pod-autoscale/

```
apiVersion: autoscaling/v2beta2
kind: HorizontalPodAutoscaler
metadata:
  name: nginx
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: nginx
  minReplicas: 1
  maxReplicas: 10
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 50
  - type: Resource
    resource:
      name: memory
      target:
        type: AverageValue
        averageValue: 100Mi
```

Update the autoscaling
```
kubectl autoscale deploy nginx --min=2 --max=10 --cpu-percent=50
```
