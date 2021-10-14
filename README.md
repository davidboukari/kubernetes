# kubernetes

# create a K3s cluster
* https://github.com/davidboukari/kubernetes/blob/main/create-cluster-k3s.md

____________________________________________________________________________________________________
## Lazy mode firewalling
iptable -F

____________________________________________________________________________________________________
## Install bash completion
```
yum install bash-completion
echo 'source <(kubectl completion bash)' >>~/.bashrc
```

## Using minikube
* https://minikube.sigs.k8s.io/docs/tutorials/multi_node/
```
minikube start --nodes 2 -p multinode-demo
minikube stop

```

### Destroy minikube clusters
```
minikube delete --all
🔥  Suppression de "minikube" dans docker...
```
____________________________________________________________________________________________________
## Initialize a kube cluster
* https://github.com/davidboukari/vmware/edit/main/kubernetes/README.md (vmare-tanzu-kubernetes-cluster doc)
```bash
yum install docker
cd ~/vmware/kubernetes/ansible-vsphere-tanzu-kubernetes && ansible-playbook playbook.yml
set the credentials (VSPHERE_USERNAME & VSPHERE_PASSWORD) in ~/.tkg/config.yaml

#tkg init --infrastructure vsphere --name my-vsphere-cluster --plan dev  --vsphere-controlplane-endpoint-ip 192.168.0.155 --deploy-tkg-on-vSphere7
tkg init --infrastructure vsphere --name my-vsphere-cluster --plan dev  --vsphere-controlplane-endpoint-ip 192.168.0.155 --deploy-tkg-on-vSphere7 --size small -v 200
...
...
Context set for management cluster my-vsphere-cluster as 'my-vsphere-cluster-admin@my-vsphere-cluster'.
Deleting kind cluster: tkg-kind-c5cpt0mrgda7j843p6m0

Management cluster created!


You can now create your first workload cluster by running the following:

  tkg create cluster [name] --kubernetes-version=[version] --plan=[plan]
...


tkg create cluster my-vsphere-cluster --plan dev --vsphere-controlplane-endpoint-ip 192.168.0.188  --size small -v 200

tkg get management-cluster
kubectl config use-context my-vsphere-cluster-admin@my-vsphere-cluster
# Scale the workers
tkg scale cluster my-vsphere-cluster --worker-machine-count 4  --namespace tkg-system
```

## Context
```
$ kubectl config get-clusters
NAME
multinode-demo

$ kubectl config get-contexts
CURRENT   NAME             CLUSTER          AUTHINFO         NAMESPACE
*         multinode-demo   multinode-demo   multinode-demo   default

kubectl config use-context my-vsphere-cluster-admin@my-vsphere-cluster
```

## Namespace
```
kubectl config get-contexts
CURRENT   NAME       CLUSTER    AUTHINFO   NAMESPACE
*         minikube   minikube   minikube   default

kubectl config set-context --current --namespace=dev
Context "minikube" modified.

kubectl config get-contexts
CURRENT   NAME       CLUSTER    AUTHINFO   NAMESPACE
*         minikube   minikube   minikube   dev
```


## Get all objects of a namespace
```
kubectl -n dev get $(kubectl api-resources --namespaced=true --no-headers -o name | egrep -v 'events|nodes' | paste -s -d, - ) --no-headers
```

```
kubectl -n dev api-resources --verbs=list -o name
```

## Shortcut
```
    Namespaces (Alias : ns) 
    Pods (Alias : po) 
    Secret (Pas d’alias…)
    Configmaps (Alias : cm) 
    PersistentVolumeClaim (Alias : pvc) 
    Persistentvolumes (Alias : pv) 
    ReplicaSet (Alias : ns) 
    Deployments (Alias : deploy) 
    Ingress (Alias : ing) 
    Services (Alias : svc) 
```

## Create a secret for the
* https://kubernetes.io/fr/docs/tasks/configure-pod-container/pull-image-private-registry/

```
kubectl -n dev create secret docker-registry regcred --docker-server=https://index.docker.io/v1/ --docker-username=myuser --docker-password=mypasswd --docker-email=mymail
kubectl get secret regcred --output=yaml^C

kubectl -n dev get secret regcred --output="jsonpath={.data.\.dockerconfigjson}" | base64 --decode
```
____________________________________________________________________________________________________
## To test commands
* Katacoda: https://www.katacoda.com/courses/kubernetes/playground


____________________________________________________________________________________________________
## Switch context
* https://github.com/ahmetb/kubectx

____________________________________________________________________________________________________
## Cluster info
```
$ kubectl cluster-info
Kubernetes master is running at https://192.168.0.155:6443
KubeDNS is running at https://192.168.0.155:6443/api/v1/namespaces/kube-system/services/kube-dns:dns/proxy
```

```
kubectl cluster-info dump
```
## node Labels
```
kubectl get nodes --show-labels

kubectl label nodes my-vsphere-cluster-md-0-686fc88ccd-582bm az=paris-fr
kubectl label nodes my-vsphere-cluster-md-0-686fc88ccd-d9g68 az=dijon-fr
kubectl label nodes my-vsphere-cluster-md-0-686fc88ccd-z4vlw az=grenoble-fr

kubectl label node my-vsphere-cluster-md-0-686fc88ccd-582bm az-
kubectl label node my-vsphere-cluster-md-0-686fc88ccd-d9g68 az-
kubectl label node my-vsphere-cluster-md-0-686fc88ccd-z4vlw az-
```

## Set node label
```
kubectl label nodes <your-node-name> disktype=ssd
```

## Dump yaml 
```
kubectl -n dev get po httpd-ihs-dijon-fr-7968964766-2wpqw -o yaml
```

## Show labels
```
root@localhost dev]# kubectl get nodes --show-labels
NAME                                       STATUS   ROLES    AGE   VERSION            LABELS
my-vsphere-cluster-control-plane-jddvh     Ready    master   11h   v1.19.1+vmware.2   beta.kubernetes.io/arch=amd64,beta.kubernetes.io/instance-type=vsphere-vm.cpu-2.mem-4gb.os-photon,beta.kubernetes.io/os=linux,kubernetes.io/arch=amd64,kubernetes.io/hostname=my-vsphere-cluster-control-plane-jddvh,kubernetes.io/os=linux,node-role.kubernetes.io/master=
my-vsphere-cluster-md-0-686fc88ccd-582bm   Ready    <none>   11h   v1.19.1+vmware.2   az=paris.fr,beta.kubernetes.io/arch=amd64,beta.kubernetes.io/instance-type=vsphere-vm.cpu-2.mem-4gb.os-photon,beta.kubernetes.io/os=linux,kubernetes.io/arch=amd64,kubernetes.io/hostname=my-vsphere-cluster-md-0-686fc88ccd-582bm,kubernetes.io/os=linux
my-vsphere-cluster-md-0-686fc88ccd-d9g68   Ready    <none>   11h   v1.19.1+vmware.2   az=dijon.fr,beta.kubernetes.io/arch=amd64,beta.kubernetes.io/instance-type=vsphere-vm.cpu-2.mem-4gb.os-photon,beta.kubernetes.io/os=linux,kubernetes.io/arch=amd64,kubernetes.io/hostname=my-vsphere-cluster-md-0-686fc88ccd-d9g68,kubernetes.io/os=linux
my-vsphere-cluster-md-0-686fc88ccd-z4vlw   Ready    <none>   11h   v1.19.1+vmware.2   az=grenoble.fr,beta.kubernetes.io/arch=amd64,beta.kubernetes.io/instance-type=vsphere-vm.cpu-2.mem-4gb.os-photon,beta.kubernetes.io/os=linux,kubernetes.io/arch=amd64,kubernetes.io/hostname=my-vsphere-cluster-md-0-686fc88ccd-z4vlw,kubernetes.io/os=linux
```

## delete label node
``` 
kubectl label node <nodename> <labelname>-
```

## Get a terminal
```
kubectl -n dev1 exec -it httpd-ihs-dijon-fr-b5677ffd4-vn9md -- /bin/bash
```

## Nodes Managements = master & Workers = none
```bash
$ kubectl get nodes
NAME                                              STATUS   ROLES    AGE   VERSION
vsphere-management-cluster-control-plane-zffw4    Ready    master   83m   v1.19.1+vmware.2
vsphere-management-cluster-md-0-bcbd45f87-l9xfp   Ready    <none>   76m   v1.19.1+vmware.2
```
____________________________________________________________________________________________________
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

## Volumes
* EmptyDir
```
apiVersion: v1
kind: Pod
metadata:
  name: nginx-volume-emptydir-dijon-fr
  labels:
    app: nginx
    env: dev
spec:
  containers:
  - name: nginx-volume-emptydir-dijon-fr
    image: nginx
    volumeMounts:
    - name: nginx-volume-emptydir-dijon-fr
      mountPath:  /data/db
    ports:
    - containerPort: 80
    env:
    - name: NGINX_PASSWORD
      value: mysecretpassword
  volumes:
  - name: nginx-volume-emptydir-dijon-fr
    emptyDir: {}
```

* Directory on the kube hosts
```
apiVersion: v1
kind: Pod
metadata:
  name: nginx-volume-directory-dijon-fr
  labels:
    app: nginx
    env: dev
spec:
  containers:
  - name: nginx-volume-directory-dijon-fr
    image: nginx
    volumeMounts:
    - name: nginx-volume-directory-dijon-fr-1
      mountPath:  /data-db-nginx
    ports:
    - containerPort: 80
    env:
    - name: NGINX_PASSWORD
      value: mysecretpassword
  volumes:
  - name: nginx-volume-directory-dijon-fr-1
    hostPath:
      path: /data-db
      #type: Directory
      type: DirectoryOrCreate
```




## Services 
* https://medium.com/google-cloud/kubernetes-nodeport-vs-loadbalancer-vs-ingress-when-should-i-use-what-922f010849e0

____________________________________________________________________________________________________
###  Kubernetes Services Objects
kube-proxy is available on each pod and it manage the Load Balancing (userspace, iptables, ipvs)

____________________________________________________________________________________________________
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

____________________________________________________________________________________________________
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

____________________________________________________________________________________________________
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

____________________________________________________________________________________________________
* ExternalName: Associate to DNS name
```

```

____________________________________________________________________________________________________
* taint untaint if already taint
```
kubectl taint node my-vsphere-cluster-control-plane-nwcwx node-role.kubernetes.io/master:NoSchedule-node/my-vsphere-cluster-control-plane-nwcwx
```

____________________________________________________________________________________________________
ConfigMap
```
# From file
kubectl create configmap nginx-config --from-file=./nginx.conf

# From env file
kubectl create configmap nginx-env --from-env-file=./nginx.env

# From literral
kubectl create configmap nginx-litteral --from-literal=DOMAIN=thedomain.com --from-literal=HOSTNAME=myhostname

kubectl get cm nginx-config -o yaml
```

____________________________________________________________________________________________________

* Use a configMap in a Pod
```
apiVersion: v1
kind: Pod
metadata:
  name: mypod
spec:
  containers:
  - name: mypod
    image: redis
    volumeMounts:
    - name: foo
      mountPath: "/etc/foo"
      readOnly: true
  volumes:
  - name: foo
    configMap:
      name: myconfigmap
```

____________________________________________________________________________________________________
* Secret
generic
```
kubectl create secret generic dev-db-secret --from-literal=username=devuser --from-literal=password='S!B\*d$zDsb'

echo -n 'admin' > ./username.txt
echo -n '1f2d1e2e67df' > ./password.txt
kubectl create secret generic db-user-pass --from-file=./username.txt --from-file=./password.txt
echo YWRtaW4=|base64 -d
admin

apiVersion: v1
kind: Pod
metadata:
  name: mypod
spec:
  containers:
  - name: mypod
    image: redis
    volumeMounts:
    - name: foo
      mountPath: "/etc/foo"
      readOnly: true
  volumes:
  - name: foo
    secret:
      secretName: db-user-pass

```
* docker-registry
```
kubectl create secret docker-registry regcred --docker-server=<your-registry-server> --docker-username=<your-name> --docker-password=<your-pword> --docker-email=<your-email>

apiVersion: v1
kind: Pod
metadata:
  name: private-reg
spec:
  containers:
  - name: private-reg-container
    image: <your-private-image>
  imagePullSecrets:
  - name: regcred

```

* TLS
```
kubectl create secret tls domain-pki --cert mycert.cert --key mykey.key

apiVersion: v1
kind: Pod
metadata:
  name: mypod
spec:
  containers:
  - name: mypod
    image: redis
    volumeMounts:
    - name: foo
      mountPath: "/etc/foo"
      readOnly: true
  volumes:
  - name: tls
    secret:
      secretName: domain-pki
```

____________________________________________________________________________________________________
## Scheduling

### By label

____________________________________________________________________________________________________
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

____________________________________________________________________________________________________
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


____________________________________________________________________________________________________
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

____________________________________________________________________________________________________
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

____________________________________________________________________________________________________
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
____________________________________________________________________________________________________
## DaemonSet
Use to be sure that some pod are present in each node. It is very use to install some agents (like metric agent, logs, ...)
```
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: www
spec:
  template:
    metadata:
      labels:
        app: web
    spec:
      containers:
        - name: www
          image: nginx
          ports:
          - containerPort: 80
          
```

____________________________________________________________________________________________________
* Job - CronJob minute (0 - 59) hour (0 - 23) day of the month (1 - 31) month (1 - 12) day of the week (0 - 6) 
```
apiVersion: batch/v1beta1
kind: CronJob
metadata:
  name: hello
spec:
  schedule: "*/1 * * * *"
  jobTemplate:
    spec:
      template:
        spec:
          containers:
          - name: hello
            image: busybox
            imagePullPolicy: IfNotPresent
            args:
            - /bin/sh
            - -c
            - date; echo Hello from the Kubernetes cluster
          restartPolicy: OnFailure

```

____________________________________________________________________________________________________
* IngressController (routage rules) diff than NodePort & LoadBalancer

IngressController routage by domain
```
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: name-virtual-host-ingress
spec:
  rules:
  - host: foo.bar.com
    http:
      paths:
        backend:
            serviceName: service1
            servicePort: 80
```

* Ingress routage by pass
```
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: name-virtual-host-ingress
spec:
  rules:
  - host: foo.bar.com
    http:
      paths:
      - path: /www
        pathType: Prefix
        backend:
          service:
            name: service1
            port:
              number: 80
```

* Ingress TLS routage
```
apiVersion: v1
data:
  tls.crt: base64 encoded cert
  tls.key: base64 encoded key
kind: Secret
metadata:
  name: testsecret-tls
  namespace: default
type: kubernetes.io/tls

apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: tls-example-ingress
spec:
  tls:
  - hosts:
      - https-example.foo.com
    secretName: testsecret-tls
  rules:
  - host: https-example.foo.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: service1
            port:
              number: 80
```




____________________________________________________________________________________________________
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
____________________________________________________________________________________________________
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

* Update the autoscaling
```
kubectl autoscale deploy nginx --min=2 --max=10 --cpu-percent=50
```
____________________________________________________________________________________________________
* Operators Resource which manage deployment, configuration, update, scaling, backup, recovery
** Operator = Controller + Custom Resource
** https://github.com/operator-framework/awesome-operators

____________________________________________________________________________________________________
* Manage kubernetes packages
https://github.com/helm/helm/releases
```
# Install repo
helm repo add stable https://charts.helm.sh/stable
# See helm app repo hub.helm.sh
helm repo add hashicorp https://helm.releases.hashicorp.com
helm install my-consul hashicorp/consul --version 0.29.0

# Create new project
helm create myfirtapp
```

## helm V2
It use the daemon tiler in the cluster
The generic repo are used

Ex: redis
https://github.com/helm/charts/tree/master/stable/redis
```
helm install stable/redis --name my-redis --set cluset.slaveCount=2

# nb slave
cluset.slaveCount=2

metadata:
  name: {{ template "redis.fullname" . }}-metrics
  labels:
    app: {{ template "redis.name" . }}
    
```
# Deploy wordpress

```
curl -O https://github.com/eazytrainingfr/kubernetes-training/blob/master/tp-5/values.yml
helm install stable/worpress --name wordpress -f values.yml
kubectl get po
kubectl get no
kubectl get svc
```

## helm V3
tiler has been removed. The command are in client side.
We should add the repositories

``` 
helm repo add bitnami https://charts.bitname.com/bitnami
helm install worpress bitnami/wordpress -h http:://.../tp-5/values.yml
kubectl get svc 

# Tp installation
# Installation
curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/master/scripts/get-helm-3
chmod 700 get_helm.sh
yum install openssl -y
./get_helm.sh
helm version

# Déploiement Wordpress
helm repo add bitnami https://charts.bitnami.com/bitnami
helm install wordpress bitnami/wordpress -f https://raw.githubusercontent.com/eazytrainingfr/kubernetes-training/master/tp-5/values.yml

# Lien utile
https://devopscube.com/install-configure-helm-kubernetes/
```

## HELM
```
# Tp installation
# Installation
curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/master/scripts/get-helm-3
chmod 700 get_helm.sh
yum install openssl -y
./get_helm.sh
helm version


helm create mychart
helm install mychart --dry-run --debug ./mychart
helm install mychart --dry-run --debug ./mychart --set service.type=NodePort
helm lint mychart
```

### create requirements.yaml for check dep
vim requirements.yaml
```
dependencies:
  - name: mariadb
    version: 0.6.0
    repository: https://kubernetes-charts.storage.googleapis.com



```


```
helm dep update ./chart
helm package mychart
helm install --name mynewinstall mychart-...tgz --set service.type=NodePort

$ helm search repo vault
NAME           	CHART VERSION	APP VERSION	DESCRIPTION
hashicorp/vault	0.16.1       	1.8.3      	Official HashiCorp Vault Chart

$ helm pull hashicorp/vault
$ ls vault-0.16.1.tgz
vault-0.16.1.tgz


$ helm search repo vault
NAME           	CHART VERSION	APP VERSION	DESCRIPTION
hashicorp/vault	0.16.1       	1.8.3      	Official HashiCorp Vault Chart

$ helm search hub vault
URL                                               	CHART VERSION	APP VERSION	DESCRIPTION
https://artifacthub.io/packages/helm/wenerme/vault	0.16.1       	1.8.3      	Official HashiCorp Vault Chart
https://artifacthub.io/packages/helm/hashicorp/...	0.16.1       	1.8.3      	Official HashiCorp Vault Chart
https://artifacthub.io/packages/helm/banzaiclou...	1.14.2

helm serve

Caution !!!!!!!!!!
Name	Old Location	New Location
stable	https://kubernetes-charts.storage.googleapis.com => https://charts.helm.sh/stable
incubator	https://kubernetes-charts-incubator.storage.googleapis.com	=> https://charts.helm.sh/incubator

helm  install nginx-deployment-dijon-fr  --namespace dev --dry-run --debug . -f values-dijon-fr.yaml

# Use a custom value file values-dijon-fr.yaml
helm  install nginx-deployment-dijon-fr  --namespace dev  . -f values-dijon-fr.yaml --dry-run --debug
helm  install nginx-deployment-dijon-fr  --namespace dev  . -f values-dijon-fr.yaml

helm list --namespace dev
NAME                     	NAMESPACE	REVISION	UPDATED                                 	STATUS  	CHART        	APP VERSION
nginx-deployment-dijon-fr	dev      	1       	2021-10-05 15:31:48.026187109 +0200 CEST	deployed	mynginx-0.1.0	1.16.0

helm uninstall --namespace dev  nginx-deployment-dijon-fr
release "nginx-deployment-dijon-fr" uninstalled


$ helm status --namespace dev nginx-deployment-dijon-fr
NAME: nginx-deployment-dijon-fr
LAST DEPLOYED: Tue Oct  5 15:36:00 2021
NAMESPACE: dev
STATUS: deployed
REVISION: 1
NOTES:
1. Get the application URL by running these commands:
  export NODE_PORT=$(kubectl get --namespace dev -o jsonpath="{.spec.ports[0].nodePort}" services nginx-deployment-dijon-fr-mynginx)
  export NODE_IP=$(kubectl get nodes --namespace dev -o jsonpath="{.items[0].status.addresses[0].address}")
  echo http://$NODE_IP:$NODE_PORT
```




