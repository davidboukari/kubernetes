# kubernetes

## Initialize a kube cluster
* [https://github.com/davidboukari/vmware/edit/main/kubernetes/README.md](vmare-tanzu-kubernetes-cluster doc)
```bash
yum install docker
cd ~/vmware/kubernetes/ansible-vsphere-tanzu-kubernetes && ansible-playbook playbook.yml
set the credentials in ~/.tkg/config.yaml

tkg init --infrastructure vsphere --name my-vsphere-cluster --plan dev  --vsphere-controlplane-endpoint-ip 192.168.0.155 --deploy-tkg-on-vSphere7
tkg get management-cluster
kubectl config use-context my-vsphere-cluster-admin@my-vsphere-cluster
# Scale the workers
tkg scale cluster my-vsphere-cluster --worker-machine-count 4  --namespace tkg-system
```

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
## Nodes Managements = master & Workers = none
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

### Pod for debugging
```
kubectl run -it test image=alpine 
apk add -u curl
```
###  Kubernetes Services Objects
kube-proxy is available on each pod and it manage the Load Balancing (userspace, iptables, ipvs)

* ClusterIP: Expose a Service port into the cluster only
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
      port: 8080
      targetPort: 80

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
      
kubectl get po,svc
NAME        READY   STATUS    RESTARTS   AGE
pod/debug   1/1     Running   1          3m33s
pod/nginx   1/1     Running   0          45m

NAME                 TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)    AGE
service/kubernetes   ClusterIP   100.64.0.1      <none>        443/TCP    9h
service/nginx        ClusterIP   100.66.27.214   <none>        8080/TCP   25m


kubectl run -it debug --image=alpine:latest -- /bin/sh
apk add -u  curl
curl http://100.66.27.214:8080 
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
...
```

* ClusterIP Port-Forward: Expose a Service port out of the cluster
```
kubectl get po,svc
NAME        READY   STATUS    RESTARTS   AGE
pod/debug   1/1     Running   1          3m33s
pod/nginx   1/1     Running   0          45m

NAME                 TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)    AGE
service/kubernetes   ClusterIP   100.64.0.1      <none>        443/TCP    9h
service/nginx        ClusterIP   100.66.27.214   <none>        8080/TCP   25m


kubectl port-forward svc/nginx 8080:8080
url http://localhost:8080
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
...

# in other term
curl http://localhost:8080
```

* Cluster-IP Proxy: Expose the API access out of the cluster (do not use it for a long time, just for debugging)
```
kubectl proxy
# in other term
curl  http://localhost:8001/api/v1/namespaces/default/services/
```

* NodePort: Expose Pod out of the cluster: by default 3000< NodePort <32767. It can be changed
```
apiVersion: v1
kind: Service
metadata:
  name: nginxnodeport
spec:
  selector:
    app: www-api
  type: NodePort
  ports:
  - port: 80
    targetPort: 80
    nodePort: 31100
    
kubectl get node -o wide
NAME                                       STATUS     ROLES    AGE   VERSION            INTERNAL-IP     EXTERNAL-IP     OS-IMAGE                 KERNEL-VERSION   CONTAINER-RUNTIME
my-vsphere-cluster-control-plane-nwcwx     Ready      master   10h   v1.19.1+vmware.2   192.168.0.55    192.168.0.55    VMware Photon OS/Linux   4.19.145-2.ph3   containerd://1.3.4
my-vsphere-cluster-md-0-686fc88ccd-dpvnx   NotReady   <none>   9h    v1.19.1+vmware.2   192.168.0.74    192.168.0.74    VMware Photon OS/Linux   4.19.145-2.ph3   containerd://1.3.4
my-vsphere-cluster-md-0-686fc88ccd-pmxkd   Ready      <none>   9h    v1.19.1+vmware.2   192.168.0.176   192.168.0.176   VMware Photon OS/Linux   4.19.145-2.ph3   containerd://1.3.4
my-vsphere-cluster-md-0-686fc88ccd-v5lvb   Ready      <none>   9h    v1.19.1+vmware.2   192.168.0.156   192.168.0.156   VMware Photon OS/Linux   4.19.145-2.ph3   containerd://1.3.4  
 
curl curl http://192.168.0.176:31100
```
* LoadBalancer: Integration with the provider
```
cat nginx-svc-loadbalancer.yaml
apiVersion: v1
kind: Service
metadata:
  name: nginxloadbalancer
spec:
  selector:
    app: www-api
  type: LoadBalancer
  ports:
  - port: 80
    targetPort: 80
    nodePort: 31200
```

* ExternalName: Associate to DNS name
```

```

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
